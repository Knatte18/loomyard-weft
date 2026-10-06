---
round: 5
focus:
  - "Card 4 changes the signature of `initState` in `internal/webstercli/verbs_test.go`: check that every test file that calls `initState` is among the card's Edit targets, and that the plan's `go vet` and `go test` would compile after the card is followed exactly."
  - "Card 4 removes sandbox scenario W2: check that every assertion in it is licensed for removal by the discussion decision record, and that no remaining comment, such as one in `internal/shuttleengine/claudeengine/modelswitch_test.go`, still cites a removed assertion."
  - "Card 7 rewords stencil text about `recover-batch` after a failed batch: check that each line of `webster-template-master.md` that states `recover-batch` as the way forward is amended or deliberately kept, and that the clauses pinned by `contracts/stencils/claims_test.go` stay verbatim."
  - "Card 7 adds a `recover-batch` refusal: check that the card decides which envelope flag the CLI returns for it, that the file mapping refusals to envelope flags (`internal/webstercli/recoverbatch.go`) is an Edit target, and that Master's failure ladder has a rung for that flag."
  - "Card 8 changes when `RunResult.Warnings` entries appear and their wording: check that every doc paragraph describing the old warning scope or text, including `internal/websterengine/doc.go`, is an Edit target."
  - "Card 11 skips the remote update after reset under `SkipPush`: check that the card states whether `SkipGit` also skips it, consistent with the fabric convention that `SkipGit || SkipPush` bypasses a push, and with the `WEFT_SKIP_GIT=1` fixture the planned CLI rows run under."
  - "Card 11 adds an executor, a banned token and a Kind: check that every comment pinning a count of executors, tokens or kinds, in `cmd/lyx/destructiveguard_test.go`, `internal/fabricengine/destroy.go` and `internal/fabricengine/doc.go`, is reworded in the card."
  - "The `cli-drive` Shared Decision in `_lyx/plan/00-overview.md` gives a reason for driving Publish below the CLI: check that the reason matches what the decision record's Publish acceptance item requires, and that the plan names what verifies that item."
---

Each entry names a site and a question; none states what the answer is.
