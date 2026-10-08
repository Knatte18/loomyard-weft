# Hub-wide limit on concurrent builds and test suites

`hub-build-test-limit` · Note · bug, loom, webster

## Seen

On 2026-10-08, from 19:34 to 20:07 CEST, Go builds and test suites ran with no break, from several sources at once.
The sources were webster implementer forks, lyx's own gates (verify, lint-comments, Preflight), the orch and a hub fix agent, and newly seeded pairs.
Nothing bounds how many run at once across the hub.
With four runs in flight, the user slice reached its 16 GiB `MemoryHigh` with swap full, reclaim thrashed, and the machine froze until a hard reset.

## Idea

A hub-wide limit on concurrent `go build` and `go test` runs: lyx's gates take a slot from a hub lock before they build or test, wait their turn instead of piling up, and log the wait.
Agents' own test commands reach the same limit through a lyx wrapper verb the stencils name.
