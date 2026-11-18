Lens: fix-proposal. I challenged the reviewer's proposals for F1 and F4 as the fixer carried them out.

**MEDIUM — F1, `claudeengine.LymBinary` and `codexengine.LymBinary` (the `Prepare` change)**
- Summary: `Prepare` now returns an error whenever `selfexec.LymBinary` refuses, which it always does under `go test`. The stand-in is set in only 5 test binaries: `claudeengine`, `codexengine`, `shuttleengines`, `orchcli` and `shuttleparity`.
- Evidence: the search found 9 other test files that build real engines through `NewFromConfig` or `shuttleengines.New`. They are `internal/treadleengine/smoke_judge_test.go`, `seatengine/smoke_seats_test.go`, `orchcli/smoke_live_test.go` (covered by the orchcli TestMain), `shuttlecli/smoke_interrupt_test.go`, `webstercli/smoke_test.go`, `burlerengine/smoke_{assist,cluster,round}_test.go` and `creelcli/smoke_llm_test.go`.
- Where it breaks: if any of these runs a real `Prepare` under its own test binary, the launch fails with `ErrTestBinary`.
- Why the gate missed it: I assume these are live-tier smoke tests that tiers 0 and 1 skip. The run covered only the touched packages, so none of the other packages' tests ran.
- Not checked: I did not open those files, so I don't know whether they set a stand-in elsewhere.

**LOW — F4, `shuttleConfigDirs` in `hookpython.go` (`preflight.BoardLymPresent`)**
- Summary: the hook now treats a hub whose Bolt checkout has no `_lym` as standalone, but the orch launch does not.
- Evidence: the landing summary says the orch wiring loads `shuttleengine.LoadConfig(fabricengine.ConfigDirs(location), "shuttle")` regardless of `BoardLymPresent`.
- Case: for such a damaged hub where a standalone state directory exists, the orch launch reads the hub layers and the hook reads the state directory. This reverses the mismatch the review described for the standalone-capable launches.
- Scope: narrow, since the two differ only when a state directory also exists.
- Other proposals: I found no wrong case for F2, F3, F5, F8 or F9. F6 and F7 are not this lens's.

Calls made:
1. `grep -rln` for real-engine constructors in `*_test.go` files, and `grep -rn "LymBinary = "` across test files. My context did not show which other test packages build real engines, or where the stand-in is set.
