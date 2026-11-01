```yaml
lens: coverage
outcome: reported
```

Coverage lens: I checked each decision, acceptance criterion and card branch for a planned row that would fail if it broke.

**MEDIUM — D4 criterion, "an unloadable claude.yaml falls back to `shuttleengine.DefaultAwaitCap` and the gate call still runs".** Only the resolver's fallback has a row. The gate's own fallback branch has none: the Warn once per call, then the call still capped at `DefaultAwaitCap`. In every gate row the helper either replaces `keepWarm` (`helperCapEnv`) or runs the real resolver over a valid template.
Evidence: card 11 Choice ("a resolver fallback is logged at Warn once per call and still caps"), card 11 Tests (helperCapEnv / real resolver), card 3 `plan:internal/shuttleengines#TestKeepWarm`.

**MEDIUM — D4 criterion, "an explicit `--cap` wins" for `lym shuttle await`.** Card 4 argues the existing `internal/shuttlecli#TestAwaitVerb` rows pin it. Those rows run outside any git worktree, where `awaitCap` also answers `DefaultAwaitCap`. A RunE that ignored `cmd.Flags().Changed("cap")` would still pass them, only slower. The RunE choice between the flag value and `awaitCap` is reached by no row.
Evidence: card 4 Intent ("The explicit `--cap` path is pinned by `TestAwaitVerb`'s existing rows"); `internal/shuttlecli/cli_test.go:261`, which runs from `t.TempDir()`.

**MEDIUM — D5 criterion, "no `--gate` or `keep_warm_max_reruns: 0` keep the short cap".** `runFront`'s cap choice has four branches with no row: no gate key, `MaxReruns` 0, a `ReadRunRecords` error, and a resolver fallback. `TestLongWait` tests only `longWait`, which takes the key and `maxReruns` as given. The one tier1 row takes the long-cap branch.
Evidence: card 12 Intent and Tests.

**MEDIUM — D2 criterion, "`gatecli.RunGate` … run[s] in-process and block[s] to [its] end".** Card 11 says `RunGate` reaches no front, but no row calls `RunGate` under a strand with a non-zero cap.
Evidence: card 11 Choice ("`RunGate` … reach[es] no front"); `internal/gatecli#RunGate`.

**MEDIUM — D2 branch "the directory resolves to no worktree", and the capped run outside every hub.**
- `frontDecision`'s no-worktree branch has no row.
- The one existing outside-hub strand row (`test_integration_test.go:1395`) is switched to `helperCapEnv=0`.
- So no row covers D3's bound for a capped call outside every hub: the runner unslotted and unregistered.

Evidence: card 11 Tests ("the rows that start the gate with `strandEnvEntry` add `helperCapEnv=0`"), card 11 Choice.

**MEDIUM — D1 criterion, "on Windows, a front joins a live runner and supersedes one whose pid was reused".** Only `TestStartReading` covers this, a unit test of the reading function with injected readers. The front's join comparison never runs with the Windows reader: every `TestGateTest_DetachedRun` row runs on Linux, and no card states a Windows-tagged row or that the criterion is left to review.
Evidence: card 10 `plan:internal/gatecli#TestStartReading`, card 11 Front.

**MEDIUM — D6, a slot-held card line waits for a slot.** The decision says the phase then names the slot wait with `slot_free: false` and the holders. No card 13 row holds the pool's slots while a card-verify capped call waits, so the card-verify `slots` answer is unreached.
Evidence: card 13 Tests rows; card 13 Intent (`Phase` writes `slots` with `Line`).

**MEDIUM — Constraint: `runDeploy` and `runRowCommand` run without the strand.** `TestCommandEnv` tests the helper alone. No row shows either runner passes `commandEnv(os.Environ())`; `TestRunDeploy_WritesTheLogAndReportsExitCodeAndTimeout` could take a `test -z "$LYM_STRAND_NAME"` row under `t.Setenv`. Dropping the call in either function fails nothing.
Evidence: card 5 Tests; `internal/battencli/deploy_integration_test.go:17`.

**LOW — D1 criterion, "after HEAD's tree … changed … starts a fresh run".** The supersede row changes only the dirty path set (a new untracked file). A HEAD tree change with the dirty set unchanged is unreached, so a join that compared only the dirty set would pass.
Evidence: card 11 Tests row 3.

**LOW — D3 criterion, "a front that finds a marked exit file, from a lease … answers 75 with `done: false` and replays nothing".**
- The lease row (1s cap, 1s grace) asserts only the abandoned mark and the freed slot; no later call reads that marked file.
- Only the signal-marked case is asserted at a front.
- The registration row ends the runner with `runproc.End`, where the criterion names pair teardown, which uses `EndSparing`.

Evidence: card 11 Tests rows "a 1s cap with a 1s grace…", "a SIGTERM to the runner…", "in a pair worktree…".

**LOW — D6 criterion, "a lease that passes while a `--wait` gate line runs … frees its slots".** The row's `lym` stub takes no slots, so slot release by the `--wait` child is not observed.
Evidence: card 13 Tests row 4.

**LOW — card 10 runner branches that no row reaches and no Intent specifies.**
- A `claim` that fails after the lease pre-check passed (the race against a front's create). The `helperLaunchEnv=none` row reaches only the pre-check refusal.
- A missing or unparsable `request.json`.
- An unknown verb.
- In card 13, a plan or card that no longer matches when the runner re-reads it.

Evidence: card 10 Runner verb; card 13 Intent ("reading the plan again in `Cwd`, matching the card id").

**LOW — D9.** `TestCheckTexts` asserts the new rule only on the refuse-raw steer. The raw go row's steer change in `Rows` is asserted by no row.
Evidence: card 15; `internal/shellguard/rows_test.go:45`.

Tool calls:
- Grep in `internal/loomcli/landingdeps.go` and `internal/landingshed` for a failure note that renders a gate call (D7 fidelity); my context did not show what the Publish note holds. It found nothing.
- Grep in `internal/loomcli` and `internal/loomshed` for Publish and `VerifyCommand` render sites, for the same reason. It found no agent-rendered gate call.
- Grep of `wiring.go` and the loomcli doc for Publish, to confirm the Publish note is no agent prompt. It is not.
