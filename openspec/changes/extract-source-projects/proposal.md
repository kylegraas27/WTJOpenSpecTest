# Proposal

## Why

An operator preparing a migration has no way to see what is actually in the source
system. Before anything can be mapped or loaded, they need an accurate, reviewable
inventory of the projects in a space — including projects nested inside folders,
which are easy to miss by eye and are exactly the ones that get dropped from a
migration.

Nothing in this system exists yet, so this change also establishes the first
working slice end to end: the operator opens the console, picks a space, and gets
back a saved inventory they can inspect.

## What Changes

- Connect to a source work-management API using operator-supplied credentials and
  verify the connection before any extraction runs.
- List the spaces the credentials can see, so the operator can choose one.
- Walk the chosen space's full folder tree and collect every project at any depth,
  recording each project's parent path so later stages can reproduce the hierarchy.
- Persist the result of each extraction as a durable, re-readable run record. Runs
  are never overwritten; re-extracting the same space produces a new run.
- Expose this over HTTP so the console can drive it: list spaces, start an
  extraction, check its progress, and read a finished run.
- Provide a minimal console view where the operator selects a space, starts the
  extraction, and reviews the projects that were found.

No behavior is removed or altered, because none exists yet. Nothing here is
**BREAKING**.

## Out of Scope

- Transforming or mapping projects to the target system's shape.
- Writing anything to the target system.
- Extracting tasks, subtasks, comments, attachments, custom fields, time logs, or
  users. This change extracts **projects only**.
- Incremental or resumable extraction. A run either completes or fails whole.
- Editing, filtering, or deleting runs from the console.
- Authentication of the operator to this tool. Single trusted local operator is
  assumed.
- Credential management beyond reading what the operator configured — no OAuth
  flow, no token refresh, no credential storage UI.

## Operator Configuration

This change introduces the first operator decisions. Each must be explicit; none
may be guessed at runtime.

- **Source API base URL** — which source instance to read from.
- **Source API token** — supplied via environment, never committed or logged.
- **Run data location** — the directory where run records are written.
- **Request page size** — how many records to request per page when walking the
  tree, so the operator can tune against their instance's limits.

## Capabilities

### New Capabilities

- `source-extraction`: Connecting to a source work-management system, discovering
  the projects within a chosen space, and persisting each extraction as a durable
  run record.
- `operator-api`: The HTTP surface the console drives — listing spaces, starting
  extractions, reporting run progress, and serving run results.
- `operator-console`: The browser view where a human selects what to work on and
  reviews the outcome.

### Modified Capabilities

None. This is the first change in the project.

## Impact

- New Python 3.11 package containing the extraction logic and a FastAPI
  application.
- New React + Vite frontend, served separately in development.
- New pytest suite. The source API is mocked throughout; no test performs real
  network I/O.
- New on-disk run directory. No database is introduced.
- Establishes the run-record shape that the transform and load stages will later
  read, so this change constrains those stages even though they are out of scope.
