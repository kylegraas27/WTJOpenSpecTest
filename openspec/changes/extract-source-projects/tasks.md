# Tasks

Both open questions in `design.md` are display-level and affect no spec, so they
are left unresolved and nothing below depends on them.

## 1. Project scaffolding

- [ ] 1.1 Create the Python package layout and pyproject with Python 3.11, FastAPI, Pydantic, httpx and pytest pinned, and verify `pip install -e .` succeeds in a clean virtualenv
- [ ] 1.2 Create the React + Vite frontend scaffold and verify `npm run build` completes with no errors
- [ ] 1.3 Add the pytest configuration and a CI-style test command, and verify it runs and reports zero collected failures on an empty suite

## 2. Source client and mocked source

- [ ] 2.1 Build the fake source fixture used by every test — synthetic spaces, nested folders, cyclic folders and multi-page responses — and verify a fixture self-test asserts it serves the shapes the later tests rely on
- [ ] 2.2 Write failing tests for connection verification covering valid credentials, a rejected token and an unreachable host, then implement the verification so all three pass
- [ ] 2.3 Write failing tests for paginated retrieval covering multi-page, single-page and mid-sequence failure, then implement continuation-token paging so all three pass
- [ ] 2.4 Write a failing test asserting a throttling response is retried with backoff and a persistent throttle ends as a failure, then implement bounded retry so it passes
- [ ] 2.5 Write a failing test asserting the configured token appears in no log record, then implement credential handling so it passes

## 3. Space listing

- [ ] 3.1 Write failing tests for space listing covering spaces returned, no spaces visible as a success, and a space missing a display name as a failure, then implement listing so all three pass

## 4. Folder tree traversal

- [ ] 4.1 Write failing tests for recursive discovery covering projects at the root, projects nested in folders, a ten-deep project and a space with no projects, then implement breadth-first traversal so all four pass
- [ ] 4.2 Write failing tests for ancestry covering a nested project recording its ordered folder path and a root project recording empty ancestry, then implement ancestry capture so both pass
- [ ] 4.3 Write failing tests for a project reachable by two paths appearing once, and for a two-folder cycle terminating, then implement the visited set so both pass

## 5. Record validation

- [ ] 5.1 Write failing tests for malformed project records covering a missing identifier, a missing name and two projects sharing an identifier, then implement Pydantic boundary validation so all three fail the run with the offending record identified

## 6. Run records on disk

- [ ] 6.1 Write a failing test asserting two extractions of the same space produce two runs with the first unchanged, then implement write-once run directories so it passes
- [ ] 6.2 Write a failing test asserting a completed run is readable after a simulated restart, then implement run reading so it passes
- [ ] 6.3 Write a failing test asserting a run whose status file is absent is not served as complete and a truncated inventory is reported unreadable, then implement status-written-last with atomic rename so both pass
- [ ] 6.4 Write a failing test asserting an interrupted run is recorded as failed with its reason and serves no inventory, then implement failure recording so it passes
- [ ] 6.5 Write failing tests asserting identical inventories across two runs of unchanged content, and unchanged ordering when the source reorders its responses, then implement sort-by-identifier before write so both pass

## 7. HTTP surface

- [ ] 7.1 Write failing tests for the spaces request covering spaces returned, rejected credentials reported distinctly from an empty list, and an empty list as success, then implement the endpoint so all three pass
- [ ] 7.2 Write failing tests for starting an extraction covering a run handle returned immediately, an unknown space creating no run, and missing configuration naming what is absent, then implement the endpoint so all three pass
- [ ] 7.3 Write failing tests for run status covering in-progress, complete and unknown-run, then implement the endpoint so all three pass
- [ ] 7.4 Write failing tests for run results covering an inventory with ancestry, a request while still running returning no partial inventory, and an empty inventory as success, then implement the endpoint so all three pass
- [ ] 7.5 Write failing tests asserting a source-unreachable failure and a malformed-data failure each reach the caller with their reason, then implement error surfacing so both pass
- [ ] 7.6 Write a failing test asserting no response body or run record contains the configured token, then implement response redaction so it passes
- [ ] 7.7 Document the configuration the operator must set — source base URL, token, run data location, page size — in the project README and verify the documented startup command runs as written

## 8. Operator console

- [ ] 8.1 Write failing component tests for space selection covering spaces listed, the extraction control disabled until one is chosen, no spaces stated plainly, and a load failure showing its reason, then implement the selection view so all four pass
- [ ] 8.2 Write failing component tests asserting an extraction starts only on explicit operator action and shows an in-progress state, then implement the start control and status polling so both pass
- [ ] 8.3 Write failing component tests for the results view covering projects shown with their folder path, an empty result stated as empty rather than an error, and several hundred projects remaining reachable, then implement the results view so all three pass
- [ ] 8.4 Write failing component tests asserting a failed run shows its reason and presents no inventory, then implement the failure view so both pass

## 9. End-to-end verification

- [ ] 9.1 Write an end-to-end test against the mocked source that selects a space, runs an extraction to completion and asserts the console-visible inventory matches the run record on disk, and verify it passes
- [ ] 9.2 Write an end-to-end test asserting a source failure partway through surfaces to the console as a failed run with no inventory, and verify it passes
- [ ] 9.3 Run the full suite and verify no test performs real network I/O, by asserting outbound connections are blocked for the duration of the run

## Workflow follow-up

- Archive the change once the implementation is reviewed and the suite passes.
- Verify the archived result lands the three capabilities in `openspec/specs/`.
