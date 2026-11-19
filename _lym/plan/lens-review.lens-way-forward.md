```yaml
lens: way-forward
outcome: reported
```

Task: review the Hardener plan's refusals, stops and recovery steps (way-forward lens), and check whether each next step it names can succeed from the state the refusal leaves.

**Findings**

1. **BLOCKING — Card 5 (`CheckCampaign` refusals 1, 4, 5 and 6), D2.**
   - The fix these refusals name, `lym board upsert <slug>` with new `run_settings`, cannot clear the refusal.
   - A task's `run_settings` reach a run only as the pair's local layer, written once at Seed-Child (decision record D9, "The local rule"). Preflight runs after the seed, so editing the board entry leaves the pair's local `hardener.yaml` as it was, and the next resume refuses the same way.
   - The working route is already in the tree: `internal/reviewbudget#Route`, for a local value, names `lym config <module> --layer local --set …` in the task worktree plus the board upsert. These refusals should name that route, then `lym loom resume`.
   - None of the six refusals names resuming the run after the write.

2. **MEDIUM — Card 3 (`LoadConfig` bound refusals) and card 14 (`loomCLI.wire` strict load).**
   - The refusals for `concurrency`, `live_attempts`, `verify_attempts` or `hard_max_bounces` below 1 name only `lym config hardener --set` from the prime. A bad value set through `run_settings` sits in the pair's local layer, where that write does not reach it.
   - Card 3 gives the unit no local value check of its own. `checkLocalDocument` (`internal/configreg/runsettings.go:173`) applies the local rule and then "the module's value checks". So a `concurrency: 0` in `run_settings` passes the seed. The strict load in card 14's `wire` then refuses every `lym loom` verb, `resume` included, with a way forward that cannot fix the cause.
   - Fix: route through `reviewbudget#Route`-style layer origins, or add a local value check so the seed refuses the value.

3. **MEDIUM — Card 7 (`NewSandbox`) with card 8 (`Reap`) and card 17.**
   - When `Clone` fails, the card says no hub is recorded, but the container it made holds the seeded bares, and the bare `.git` directories stay on disk.
   - `Reap` "removes the root's hub directories", and its tracking is the hub list, so an unlisted container is never removed.
   - So `clear` succeeds, and the worktree removal then fails anyway: fabric's leftover cleaner refuses a path with a nested `.git` (`internal/fabricengine#ignoredTaskPathDeletable`).
   - Card 17's way forward, re-running `lym harden sandbox clear`, cannot clear it either.
   - Fix: `Reap` removes every entry under the root, and `NewSandbox` removes its container on failure.

4. **MEDIUM — Card 9 (`WrapRow`).**
   - A reap error, a round-file write error and an `ArchiveRound` commit error are each "joined to the wrapped call's error". The card names no way forward for the halt this causes.
   - On a fix row, an archive commit failure after a successful round makes the row return an error. On resume the fix row runs again, but card 9 says no archive is retried for a round already fixed.
   - Fix: name `lym loom resume` (and, for a reap failure, `lym harden sandbox clear`) in the joined error, and say whether a resumed fix row re-archives.

5. **LOW — Card 17 (`RemovePair` sandbox-clear failure).**
   - The way forward is "run `lym harden sandbox clear` in <worktree>, then remove the pair again". It names no command for "remove the pair again", and the right one depends on the route: `lym fabric remove` for the fabric route, `lym loom resume` for batten's Worktree-Teardown row.
   - The clear runs after `t.archive` and `t.closePR` (`internal/pairteardown/teardown.go:~455-471`). Both are void and look safe to repeat, so the retry itself can progress.

6. **LOW — Card 10 (`OnHalt`).**
   - `escalateWith` serves every escalation cause (circling, budget, hard-cap, decision, stalemate), and the card calls `OnHalt` from inside it. So the sandbox is reaped on a decision or stalemate escalation too, not only on a capped or circling halt as D5 states.
   - The `OnHalt` failure text names `lym harden sandbox clear`. That command touches the same root, so this does no harm.

7. **LOW — Card 19 (`DropRetiredRoles`).**
   - The reinterpretation rewrites the hub's base `agents.yaml` only: `hubreconcile` reads `configengine.ConfigFile(boardDir, …)`.
   - A worktree or local `agents.yaml` layer that names `treadle-judge` would still fail `agentconfig.Load`. Its refusal ("names …, which is no role lym starts") names no way forward.

8. **LOW — Card 6 (live gate `Run` error).**
   - A `Run` spawn failure, or a gate-config refusal from `lym gate test`, comes back as an error, which halts the row. The card names no next step for it.
   - Preflight covers the missing-key case, but not a key that is malformed or was edited after Preflight.

**Tool calls**
- Grepped `PassOnCap` in `internal/shuttleengine` to confirm a PassOnCap entry lets the run through after its attempts run out. My context had only the summary line.
- Read `internal/pairteardown/teardown.go` `RemovePair` to check what runs before the planned clear and whether a retry repeats side effects. My context had only the agent's ordering summary.
- Grepped `internal/configreg/runsettings.go` to see what `PrepareRunSettings` validates at seed. My context did not say whether a module's value checks run there.
