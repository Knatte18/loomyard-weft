# Wire landing seams before pr-findings builds its shed

`lym loom pr-findings` failed on every findings file with `landingshed: NewFinalize: Deps.OpenFabric must not be nil`.
The verb reads the PR-Review row's send-back edge from the run's shed, and it built that shed without wiring the landing seams.
The recipe's landing rows, Finalize among them, refuse to build with those seams unset.
The verb now wires them first, so the findings round runs.

Closes #155

## Changes

- `internal/loomcli/prfindings.go`: the edge closure moves into `(*loomCLI).prReviewSendBackEdge`.
  After it finds the recipe's PR-Review row, it calls `wireLanding` and only then `newShed`.
  A recipe with no PR-Review row still answers that the edge cannot fire, without wiring anything.
- `internal/loomcli/prfindings_integration_test.go`: a new test runs `pr-findings` through the real cobra tree in a real hub pair.
  It feeds a BLOCKING findings file on a tapestry run and asserts the answer carries `send_back: true`.
- `internal/loomcli/doc.go` and the comment in `wireLanding`: both name the send-back edge as a third path through `wireLanding`, beside `run` and `step`.

## Design decisions

- The fix wires the seams at the edge, not in the shared shed construction, so `status` and `pause` still never pass through `wireLanding`.
- `wireLanding` also opens the sampler row's watch on its record.
  The edge's process exits right after it reads the shed, so that watch is never waited on.

## To check

- The regression test must reach the verb itself, not a helper beside it.
  An earlier version of the test bypassed the verb; it now calls the verb through the cobra tree with `clihelp.ExecuteIn`.
  Confirm it fails when the `wireLanding` call in `prReviewSendBackEdge` is removed.
- Nothing was run by hand against a live darn run.
