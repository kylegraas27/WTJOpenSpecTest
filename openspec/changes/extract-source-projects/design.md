# Design

## Context

Greenfield: no code exists yet. See `proposal.md` — Why for motivation, and the
three spec deltas for the behavior contract.

Constraints that shape everything below, from project context: Python 3.11,
FastAPI, Pydantic, Pytest, React + Vite; no database, state lives in JSON files on
disk; extraction is the only pipeline stage in scope and it is the stage that
touches the network; runs must be deterministic; external APIs are mocked in all
tests.

Two properties of the source system drive the design. Its folder tree is a
*graph*, not a tree — a project can be reachable by more than one path, and
folders can reference each other cyclically. And its listing endpoints paginate,
with per-request record limits.

## Goals / Non-Goals

**Goals:**

- A run is a single immutable artifact on disk, written once, readable forever.
- The network-touching code has one entry point, so later stages inherit a tested
  pagination and retry story rather than reinventing it.
- Determinism is a property of the stored artifact, not of the source's response
  order.

**Non-Goals:**

- Concurrency across runs. One extraction at a time is sufficient for a single
  local operator.
- A general source-system abstraction. A second source system is speculative; the
  seam is drawn, not generalised.
- Streaming results to the console as they are discovered. Poll-to-completion is
  enough at this size.

## Decisions

### Walk the graph iteratively with an explicit visited set

Chosen: breadth-first traversal driven by an explicit queue, with a set of folder
identifiers already visited.

*Alternative rejected:* natural recursion over children. Rejected because the
source's folder graph can contain cycles (spec: *Repeat-visit and cycle safety*),
which makes unbounded recursion a stack overflow rather than an error, and because
deep nesting has no documented depth limit.

Ancestry is carried along the queue rather than reconstructed afterwards, so the
first path on which a project is reached is the ancestry recorded. Because
traversal order is deterministic, that first path is stable between runs.

### Sort the inventory by source identifier before writing

Chosen: the walk collects, then the inventory is sorted by the project's source
identifier immediately before the run record is written.

*Alternative rejected:* preserving discovery order. Discovery order depends on the
order the source returns children, which is not contractually stable — the spec
requires identical ordering when the source returns the same projects in a
different sequence. Sorting at write time makes determinism a property of the
artifact, so it cannot be broken later by a traversal change.

*Alternative rejected:* sorting by name. Names are not unique and are
locale-sensitive to sort.

### Run records are write-once directories named by a sortable run identifier

Chosen: each run gets its own directory under the configured run location, named
with a timestamp-plus-random identifier that sorts chronologically. The inventory
and the run's status are written as separate files, and the status file is written
last.

*Alternative rejected:* a single append-only runs file. Rejected because concurrent
readers and a partially written record are hard to tell apart without a database,
and because the spec requires a corrupted run to be detectable as unreadable
rather than partially served.

Writing status last is what makes *Failed runs are distinguishable from complete
runs* enforceable: a run directory with no completed status file is not a complete
run, whatever else it contains. Each file is written to a temporary name and
renamed into place, so a crash mid-write leaves no file that looks finished.

### Fail the run on malformed source records

Chosen: a project record missing an identifier or a name aborts the run.

*Alternative rejected:* skipping bad records and reporting them as warnings.
Rejected because of the project's determinism principle — a silently skipped
project is a project missing from the migration, discovered much later. The
operator decides what to do; the tool does not guess.

### Validate source payloads at the boundary with Pydantic

Chosen: responses are parsed into Pydantic models at the point of ingest, before
traversal logic sees them.

*Alternative rejected:* passing raw dictionaries inward and checking fields where
used. Rejected because the malformed-record requirements need one place that
decides a record is unusable, and because the run record's shape then has a single
declared source of truth that the transform stage can later read.

### The API returns a run handle immediately; the console polls

Chosen: starting an extraction returns a run identifier at once, and the console
polls the run's status until it is terminal.

*Alternative rejected:* a synchronous request that returns the inventory. A walk
over a large space exceeds a comfortable request timeout, and the spec requires
progress to be observable while the run executes.

*Alternative rejected:* websockets or server-sent events. More machinery than a
single local operator needs; polling can be replaced later without a spec change.

### Credentials come from the environment, never from a request

Chosen: base URL and token are read from environment configuration at startup. No
request carries a credential, and the token is never written into a run record,
response, or log line.

*Alternative rejected:* letting the console post credentials per extraction.
Rejected because it puts the token in request bodies and browser memory for no
benefit at this scale, and makes *Credentials are never disclosed* harder to hold.

## Risks / Trade-offs

- **Source rate limits are not yet known** → All source calls go through one
  client with a configurable page size and a bounded retry with backoff on
  throttling responses. If the real instance rejects the default page size, the
  operator changes configuration rather than code.

- **Pagination contract is assumed, not verified** → Design assumes
  continuation-token pagination where the absence of a token means the last page,
  and that a page can legitimately be empty. The mocked source in tests encodes
  this assumption explicitly, so if the real API differs, the tests say where.

- **Sorting the inventory requires holding the whole space in memory** → Acceptable
  for spaces of the size a single operator migrates. A space large enough to
  exhaust memory will fail loudly rather than silently truncate, and streaming
  with an external sort is the fallback if that ever happens.

- **A long walk gives no intermediate feedback** → Status reports in-progress but
  not how far along. Counting folders visited is cheap to add later and needs no
  spec change, so it is deliberately left out of the first slice.

- **Write-once runs accumulate on disk forever** → No retention policy in this
  change. Runs are small, and a retention decision belongs with the operator once
  there is more than one stage producing them.

- **A single-path ancestry loses information when a project is reachable twice** →
  The spec requires the project to appear exactly once, so one ancestry must be
  chosen. Deterministic traversal makes the choice stable, but a project genuinely
  filed in two places will look filed in one. Recording all paths is a spec change,
  not a design change, and is deferred until the transform stage shows it matters.

## Migration Plan

Not applicable: no existing deployment, no data to migrate, nothing to roll back to.
Rollback is reverting the change.

## Open Questions

- Whether the console should offer to re-run an extraction directly from a finished
  run's view. Pure addition to the console; affects no spec in this change.
- Whether run identifiers should be exposed to the operator as-is or given a
  friendlier label. Display concern only.
