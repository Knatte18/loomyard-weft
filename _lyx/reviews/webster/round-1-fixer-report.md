# Webster-Review round 1 — fixer report

All eight findings are fixed, one commit each, none pushed.

| Finding | Commit | Change |
|---|---|---|
| F1 (BLOCKING) | `83ea35ddb` | `internal/orchcli/refusal.go`: "the prime's weft sibling" becomes "the prime's fabric sibling", battencli's phrasing, so `TestEnforcement_FabricVocabulary` passes again. |
| F2 (MEDIUM) | `3f27a75f6` | Semantic line breaks in the new comments of `shuttleengine/engine.go`, `session.go`, `session_test.go`, `claudeengine/session.go` and `claudeengine/usage.go`. |
| F3 (MEDIUM) | `96fa447a9` | Semantic line breaks in the new orchengine comments: `doc.go` (rewritten in full, content unchanged), `config.go`, `template.go`, `prompt.go`, `launch.go`, `state.go`, `watcher.go`, `watchloop.go`, `seam_enforcement_test.go`, `config_test.go`. |
| F4 (MEDIUM) | `b2a631453` | Semantic line breaks in the new orchcli comments: `cli.go`, `paths.go`, `refusal.go`, `start.go`, `status.go`, `stop.go`, `cycle.go`, `watch.go`, `spawn.go`, `session.go`, `handover.go`, and the test files `cli_test.go`, `start_test.go`, `watch_test.go`, `prime_integration_test.go`, `testmain_test.go`, `testmain_integration_test.go`, `smoke_cycle_test.go`. The `prime_integration_test.go` header also had broken grammar ("`status` and `start` run … refuses"), now fixed. |
| F5 (MEDIUM) | `ab84b3b88` | `smoke_cycle_test.go`: a new `turnEnds` helper counts Stop/Waiting events through `claudeengine.ParseEvents`. The background-task wait now requires the count to pass its pre-send value, not just any transcript path. |
| F6 (MEDIUM) | `faa1ef82f` | `smoke_cycle_test.go`: a new `askAgentAddress` sends a turn asking the session to call ListAgents and answer on one `AGENT-ADDRESS:` line, then reads the answer from that turn's Stop payload. It runs before the background task and again after the cycle, and both answers plus whether they match are logged. The background sleep is now 120s so its notification lands after `/clear`. The notification check now waits until the sleep's deadline plus 20s and looks in both the resumed transcript and the pane. |
| F7 (LOW) | `8c05ef77a` | `orchengine/doc.go`: a handoff or resume timeout (or a question during the handoff) aborts to idle, and a clearing timeout moves on to resuming. |
| F8 (LOW) | `944226426` | `orchengine/watcher_test.go`: new `TestWatcher_EveryAbortReturnPersistsCursor` covers the handoff ask abort, handoff timeout and resume-render abort. It checks `LastInjectionOffset` equals the events read through. |

Verification after the last commit:
`go build ./...` and the full untagged `go test ./...` pass, including `internal/lyxcwd`.
The plan's `integration`-tagged test set passes.
`go vet -tags smoke ./internal/shuttleengine/... ./internal/orchcli/...` is clean, and `gofmt -l` reports nothing.
The smoke test itself was not run: it needs a live, logged-in Claude Code session, so the F5 and F6 changes are compile- and vet-verified only.
For the same reason, `orchengine/doc.go`'s open risks still say "unverified, pending a smoke run".

## Deferred

Nothing deferred.
