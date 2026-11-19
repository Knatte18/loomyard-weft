# pr-findings: wire the landing seams before reading the send-back edge

`lym loom pr-findings` failed on every valid findings file with
`loomrecipe: shedbuild: producer 13 "Finalize": landingshed: NewFinalize: Deps.OpenFabric must not be nil`.
To learn whether the PR-Review row's send-back edge can fire,
the verb builds the run's shed and reads the row's reroute target from it.
It built that shed over an `Env` whose `Landing` deps were never filled.
Only `wireLanding` fills them, and only the `run` and `step` building paths called it.
So the landing rows' constructors refused to build.

## Change

- `internal/loomcli/prfindings.go`: the edge closure becomes the method `prReviewSendBackEdge`.
  It calls `wireLanding` before `newShed`, the same order `buildLoomShed` uses.
  It does not reuse `buildLoomShed`, which also ties the shuttle runner to a loop step
  and opens a pawl run's land-request watch.
  pr-findings is no step and needs neither.
  A recipe with no PR-Review row still answers without opening the fabric.
- `internal/loomcli/doc.go`: names pr-findings among the callers of `wireLanding`,
  and says the verb reads the edge from a shed built with the landing seams wired.

## Tests

`TestPRReviewSendBackEdge_BuildsTheShedOfAWiredRun` (tier 1, new) wires a tapestry run over a real hubforge pair.
It asserts that `prReviewSendBackEdge` answers true with no reason and no error.
Without the `wireLanding` call, the edge builds the shed from an `Env` that has no landing deps, the path the reported error came from.
The unit tests of `prFindingsVerb` inject a fake edge, so they could not catch the missing wiring.
