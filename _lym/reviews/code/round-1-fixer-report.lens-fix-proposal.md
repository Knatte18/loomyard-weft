```yaml
lens: fix-proposal
outcome: reported
```

Task: fix-proposal check of the review proposals the fixer carried out.

1. **MEDIUM**: F1, `teardownPushSeams` in `internal/battencli/wire.go` ("With git or pushes off … the pair reads level").
   - The proposed change contradicts `pattern/PATTERN-pair-remote.md:94-95`: "Worktree-Teardown pushes the pair to completion with the prime lock released, and removes it only while origin's tracking refs hold both tips."
   - Case: a run with `FABRIC_SKIP_PUSH=1` and unpushed pair commits now reaches `Remove` while origin holds neither tip. The PATTERN states no exception for pushes off.

2. **MEDIUM**: F3, `c.startPusher = func() { startRunPushers(…) }` in `internal/battencli/wire.go` and `startRunPusher` in `internal/loomcli/wiring.go`.
   - The tests call the extracted helpers directly, so deleting either `c.startPusher = …` assignment in `wire`, or the helper call inside it, still passes every test. The finding's premise was exactly this, and its proposal asked to assert that "the closure passes" the targets.
   - The existing resume tests replace the seam with their own (`continue_pass_test.go:76,193`, `loomcli/resume_test.go:290`), and `arm.go` skips a nil `startPusher`.
   - Similarly, in `beamcli` only `Tick` is driven. A `Runs` closure that passed `runpusher.StartIfAhead` instead of the injected `startPusher` would pass.

3. **LOW**: F1 bound and skip, `internal/battenshed/teardown.go`.
   - `internal/battenshed/doc.go:112` still says "A tip that moved since the push releases the second hold and pushes again", with no bound.
   - `deps.go:384` still says "A false read means a tip moved since the push, so the teardown pushes again". It names neither the three-push stop nor that a push that landed nothing also reads false.

4. **LOW**: F2, `TestTeardownPushError`.
   - The review's "alternatively" branch (extract the mapping only) leaves the other half of the finding's premise open: the real `PushRun` and `TipsOnOrigin` calls for a present pair.
   - A `push` that returned nil without calling `PushRun`, plus a `level` that always returned true, fails none of the new tests. The only present-pair test of `teardownPushSeams` runs with pushes off.

5. **LOW**: F4, `internal/runpusher/seams_test.go`.
   - The finding's summary lists the "NewDeps conditions" among the untested things. The proposed fix omitted them and the new file does too.
   - A `Conditions` closure in `run.go` that ignored the refused-push record or the hand-over mark is still caught by none of the added tests.

I found no concrete case against the proposals for F5, F7, F8 or F9.

Tool calls (2 of 6):
- grep of PATTERN, `pattern/`, docs and the resume tests for teardown/push rules and `startPusher` overrides: my context had no PATTERN text and no `continue_pass_test.go`.
- `sed` of `PATTERN-pair-remote.md:88-108` and grep of `battenshed/doc.go` and `deps.go`: my context had neither the entry's full sentence nor those two files.
