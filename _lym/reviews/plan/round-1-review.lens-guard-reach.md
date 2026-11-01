```yaml
lens: guard-reach
outcome: reported
```

**guard-reach: each guard, refusal, branch or test row the cards add, and whether it can be reached**

- C1 `LoadConfig` refuses a negative `keep_warm_max_reruns`: reachable through a local layer (the key is `CategoryNone`); the `-1` row reaches it.
- C2 `ValidateKeepWarm` (no parse, below 60s, at or above the lifetime, above 55m), in `Prepare`, `KeepWarmCaps` and the claude `CheckLoad`: reachable through claude.yaml layers, and rows reach each. The TTLs accept only `5m` or `1h` (`command.go` `validatePromptCacheTTL`), so the 250s template never trips it.
- C3 fallback branches (strand does not parse, shuttle.yaml load fails, set does not build, `KeepWarmCaps` error): all reachable. `engine` is `CategoryNone` and `LoadConfig` checks only that it is non-empty (`shuttleengine/config.go:75,110`), so `engine: bogus` loads and reaches the build-failure branch rather than the load branch.
- C3 numbered-role rule through `baseRole`: reachable (rows `implementer-2` and `implementer-x`).
- C4 `awaitCap` branches (flag given, worktree found, no worktree): reachable, and `TestAwaitCap` rows reach each.
- C5 strand removal in `runDeploy`, `runRowCommand` and `runLine`: reachable. No other production Go site execs `lym gate test` without `--wait` (search over `internal`/`cmd`); `loomshed/plancontext.go:143` keeps Verify lines on `--wait`.
- C6 `RunTied` with the parent-death signal and the tree record: reachable on Linux (helper SIGKILL row, `KillTree` row).
- C7 cancel during the slot wait and during the line: reachable (rows). `Phase` and `Tied` hooks: reachable.
- C9 `Abandoned` set in `writeRecord` when `e.ctx` is cancelled: reachable (signal row). A tier `timeout_min` or the pre-tier ceiling does not cancel `e.ctx` (`run.go` uses `runCtx`), so neither marks a record.
- C10 runner refusals (no lease, passed lease, missing or bad request, bad verb, already claimed): reachable only by a direct call or a front's own create, and each has a `TestGateRunner_Refusals` row. The claimed row reaches the claim because the lease and request checks pass first.
- C10 runner lease cancel in the slot wait and in a tier, signal giving `{143,true}`, phase states: reachable (`TestGateRunner` rows).
- C11 `frontDecision` (`--wait`, empty strand, no worktree, zero cap): reachable, with rows.
- C11 inherited-slot branch: reachable in production only by a hand-set slot variable beside a strand. Every lym path that holds a slot strips the strand: the tier env at `run.go`, plus `verifytree`, card 5 and the runner. The card states this bound, and the "inherited held slot" row reaches it, since the unparsed `gate-test-strand` resolves the fallback, a non-zero cap. Not a finding.
- C11 lock-margin answer, claim-bound start failure, launch-error start failure, late claim, replaying 75 as 1, marked exit, front SIGTERM, registration and prime non-registration: reachable, with rows.
- C11 `launchRunner` refusing under `testing.Testing()`: no reaching test. It is a fail-loud wiring guard for the Constraint that tests never re-exec `os.Executable()`, so it is exempt. The test helpers that keep the real resolver with a strand all take the inherited branch or set `helperCapEnv=0`.
- C12 `chooseCap` and `longWait` branches: reachable (`TestChooseCap`, `TestLongWait`, and the integration row with seeded records).
- C13 refusal when the runner's plan re-read or card match fails: reachable (late-launch row with the plan rewritten).
- C13 supersede of a dead card-verify runner killing the `line.json` group: reachable. The backgrounded child keeps the pgid and starts after the record, so `KillTree`'s start check passes (`tied_linux.go:57-82`).
- C13 Windows `HoldNamedJob` failure: a Warn branch, not a refusal; Windows only, and no test.
- C13 runner re-reads a card whose `**Verify:**` line was removed: the state can occur but no branch handles it. Finding 1.
- C11 row `helperLaunchEnv=none` asserting the runner's log on stderr: no runner writes `runner.log` in that row. Finding 2.

**Findings**

1. LOW · consistency · card 13 (`plan:internal/gatecli#runDetached`, card-verify branch). The runner re-reads the plan and handles only "plan read fails" and "card match fails". A card that still matches but whose `**Verify:**` line was deleted between the front's read and the runner's (the same edit window the card's late-launch row uses) reaches no stated branch. The in-process path answers ok with "has no **Verify:** line" (`internal/gatecli/cardverify.go:95-97`); the runner's exit code and replay for that state are unspecified. Evidence: `_lym/plan/13-card-verify-capped.md` Intent ("When the plan read or the card match fails in the runner…"); `internal/gatecli/cardverify.go:95-97`.
2. LOW · consistency · card 11 (`plan:internal/gatecli#TestGateTest_DetachedRun`, row `helperLaunchEnv=none`). The row asserts the call "exits 1 with the runner's log on stderr". Under `none`, `helperLaunch` returns the pid of an already-ended process and starts no runner, so nothing writes `runner.log`, and the assertion can only see an empty or absent log. The row should name what the ended process writes to `runner.log`, or assert the start-failure envelope alone. Evidence: `_lym/plan/11-gate-capped-front.md` Tests paragraph ("`none` returning the pid of an already-ended process so the runner never claims") and the row "`helperLaunchEnv=none` … exits 1 with the runner's log on stderr".

Tool calls:
- `internal/shuttleengine/claudeengine/command.go:136-168`, plus a search for production and test sites that exec `lym gate test` outside gatecli: the context lacked the TTL value set (to judge whether the 250s cap can trip the C2 validation on an existing config) and lacked any `--wait`-less Go exec site of the gate.
- `internal/shuttleengine/config.go`, the `engine` key's category and validation: the context lacked whether `engine: bogus` reaches C3's build-failure branch or the load-failure branch.
