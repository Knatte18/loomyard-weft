---
verdict: APPROVED
rationale: "All eight round-1 findings are fixed in commits 83ea35ddb..944226426; full go test ./... (incl. internal/lyxcwd), the integration set and go vet -tags smoke pass, and no new comment breaks mid-sentence"
---

Review range: `_lyx/shed/hub-orch-strand/status.json` names `product.parent` `main`, so the range is `git diff $(git merge-base main HEAD)..HEAD`: cards 1–14 plus eight fix commits.

The round-1 report carries no `## Focus departures` section, so there is nothing to ratify or reject.

Each finding checked against the current HEAD:

- **F1** (BLOCKING, fabric vocabulary): `refusal.go` now says "fabric sibling".
  A full untagged `go test ./...` exits 0, `TestEnforcement_FabricVocabulary` included.
  The remaining `weft` tokens in `orchcli` are in `prime_integration_test.go`, a test file the tree scan does not cover, and match battencli's test files.
- **F2–F4** (comment line breaks): a scan of every added `//` line in the range for a comment line continuing onto the next without a sentence end finds only breaks at independent-clause boundaries (", and …", ", so …" with their own subject and verb).
  That is what CONSTRAINTS.md's Comment Line-Break Convention allows.
- **F5** (smoke background-task wait): the wait now needs `turnEnds` to pass its pre-send count, so it cannot succeed on the start prompt's own Stop.
- **F6** (smoke open-risk observations): `askAgentAddress` asks for the ListAgents name on a marked line before and after the cycle and logs both answers and whether they match.
  The notification check waits until the sleep's deadline plus a margin, then reads both the resumed transcript and the pane.
  `EventStop.Message` carries `last_assistant_message`, so the answer really can be read back.
- **F7** (doc.go clearing timeout): the package doc now separates the handoff and resume timeouts, which abort to idle, from the clearing timeout, which moves on to resuming.
- **F8** (cursor on abort returns): `TestWatcher_EveryAbortReturnPersistsCursor` covers the ask abort, the handoff timeout and the resume-render abort, and asserts idle, `LastAbortReason` and `LastInjectionOffset`.

Substrate re-run at HEAD: `go build ./...`, `go test ./...`, `go test -tags integration` over orchcli/orchengine/shuttleengine/cmd/lyx, `go vet -tags smoke` and `gofmt -l` are all clean.
The per-card Edit-group blast-radius checks the round-1 review ran still hold: the fix commits touch only comments, `smoke_cycle_test.go` and `watcher_test.go`.

A residual note that does not block: after the sleep deadline, a background-task notification can start a turn of its own.
That turn end could satisfy the second `askAgentAddress` wait before the address turn ends, and the logged answer would then be empty.
It is an observation log, not an assertion, and an empty answer is logged as `stable=false`, not as a false pass.
