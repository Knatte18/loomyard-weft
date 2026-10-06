---
round: 3
focus:
  - "Reset's remote update (the plan's cards on `UpdateRemoteBranchLeased` in `internal/gitrepo` and on `ResetPairWarp` in `internal/fabricengine`): can `ResetPairWarp` tell a rejected lease from any other fetch or update failure without matching git's stderr outside `internal/gitrepo`, and does a test reach each of the two refusals the decision record gives (moved remote tip, remote unreachable or update failing)?"
  - "Every new `*gitrepo.Repo` method that calls `r.run` or `r.runChecked`, and every new fabric executor that pushes or deletes a remote branch: does some card edit the pinned lists in `cmd/lyx/gitrepoboundary_test.go` and `cmd/lyx/destructiveguard_test.go` so `go test` passes at that card, and does the plan's `## verify:` section run `cmd/lyx`?"
  - "Existing tests that build a plan or a state a new refusal or check now rejects (for example plans where one card `Uses` a later card's target, run through `websterengine.Run`): does the card that adds the check name and edit or delete each such test, rather than leave it to a grep?"
  - "Each card that adds a check ID or a refusal: do the spec's validation-checks list and the owning package's `doc.go` both get a row in that card, as the decision record's constraints require?"
---

Entries restate their own sites so a fresh reviewer needs no earlier round.
