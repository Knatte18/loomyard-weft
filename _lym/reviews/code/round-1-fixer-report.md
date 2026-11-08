## Changes

- F1 (fixed as proposed): the seven doc comments the review named now call the presence the machine's (`orchcli/paths.go`, `orchengine/state.go`, `orchengine/prompt.go`, `orchengine/doc.go`, `dobbyengine/waitreminder.go`).
  The two test comments in `orchcli/start_test.go` and `orchengine/watcher_test.go` that said the same were reworded too.
- F2 (fixed as proposed): `promptHook` logs the failed creel read and `replyHook` logs the failed post with `logger.Warn`, the strand and the error.
  The hooks' other failing steps (presence read, presence write, orch client lookup) log through a new `warnHookFailure`, after the self-check showed the same gap.
- F3 (fixed as proposed): `internal/reedengine/testmain_guard_test.go` re-executes the test binary with one positional argument and asserts exit status 2, empty stdout and the refusal on stderr.
  The child carries `REEDENGINE_TESTMAIN_GUARD_CHILD`, which makes the test skip itself, so a missing guard fails the exit-status check instead of recursing.

Gate repair: the verify run failed `TestHermeticGitEnv_GitSpawningPackagesHaveTestMain`, because the F3 test's `exec.Command` marks `internal/reedengine` as git-spawning.
`cmd/lym/hermeticenv_test.go` now allowlists `internal/reedengine/testmain_guard_test.go`, beside the package's other non-git spawn entry.
`lym gate test ./cmd/lym ./internal/reedengine` passed at tier 0.

Gate: `lym gate test --tier 1` over `slackcli`, `reedengine`, `orchcli` and `orchengine` passed.

## Self-check

- fix-code-correctness, MEDIUM: the guard test recurses without the guard. Fixed with my own change as the lens proposed (child env var and skip).
- fix-consistency, LOW: two test comments still said the hub's presence. Fixed.
- fix-affected-set, LOW (same finding): merged with the one above.
- fix-affected-set, LOW: the presence read and write and the orch client lookup in the hooks logged nothing. Fixed with `warnHookFailure`.
- fix-proposal, LOW: same hook gap as above and the same stale comments. Merged and fixed.
- fix-proposal, LOW: the guard test covers one positional argument only. Not fixed: the lens found no wrong result, and the guard tests `flag.NArg() > 0`, so one argument pins it. Disputed below.

## Disputed

- fix-proposal LOW (one-argument guard test): ground `premise false`. The lens itself names no wrong-result case, and `testmain_test.go:21` tests `flag.NArg() > 0`, not the first argument alone.

## Deferred

None.

## Rulings

None.
