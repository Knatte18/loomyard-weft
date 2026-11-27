# research: test, build and wall time of every package's tests

Adds `docs/research/speed/test-time-measure-2026-11-27/`: `report.md`, with one table of every package that holds tests, and `calls.tsv`, with the figures and machine load of each of the 315 test runs.
It is indexed in `docs/research/README.md`.
No test or production code changes.

The report says where test time goes, so that the packages to cut or restructure can be picked:

- `internal/fabricengine`, `internal/battencli` and `internal/gatecli` have the longest integration test times (142 s, 75 s and 56 s on the first run).
- Packages with 15 s or more of test time are each given a kind of change: restructure, split long scenarios, share fixtures, drop sleeps or run in parallel.
- An integration run pays about 12 s of build and vet before any test, which exceeds the test time in 72 of the 82 packages that have such tests.
- Every run passed.

Design decisions and limits:

- Each package and level ran once, and the ten slowest by wall time ran twice. Each run was forced, with no earlier pass reused.
- Wall time starts when the run gets its concurrency slot, so time queued behind other runs is not counted as build time.
- The machine was heavily loaded (1-minute load median 45 on 12 threads during the integration runs), so absolute times are inflated.
  `internal/battencli` took 75 s and then 33 s.
- Runs were made by package, so tests normally held back for particular checks ran too, in `cmd/lym` and `internal/gatecli`.
- The suggested causes in `internal/boardengine`, `internal/fabriccli` and `internal/pairteardown` (a push backoff waited out in real time) are inferred from durations and code reading, not timed directly.
- The fixed build steps were not timed one by one.

## Merge danger

**Door:** two-way
**Blast radius:** docs
