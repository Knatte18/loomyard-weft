```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens: I traced whether each guard, refusal and test row the cards add can be reached, and whether a guarded site recurs unguarded.

**Findings**

- **MEDIUM. Card 16 (`clear`), D5 and D11: the `clear` verb's reap has no wiring it can reach.**
  - Card 16 has `clear` run `hardenshed.Reap` "with the seams card 14 wires".
  - Card 14 builds those seams inside `plan:internal/loomcli#hardenCampaign`, which is unexported and lives in loomcli. `internal/hardencli` cannot call it.
  - No card creates a shared, exported builder of the reap seams (`proc.ListProcesses`, `runproc.Owners`/`End`/`EndProcess`, `reedengine.StopServer` with reed's tmux path).
  - So `clear`'s guards and its bound have no production path. This includes the teardown route of card 17, which spawns `clear`. Evidence: card 14 Create `plan:internal/loomcli#hardenCampaign`; card 16 Intent "with the seams card 14 wires".

- **MEDIUM. Card 6 (`live` gate), D4: the started-no-binary guard reads only a printed started line, and a passing call with no such line has no stated meaning.**
  - Card 6 step 3 reads started counts through `gateslot.ParseStartedLines`, and step 4 marks red when `harden-live` "started no test binary".
  - Nothing says how a passing call whose output carries no started line counts: absent and zero are not told apart.
  - `internal/gatecli/tierrun.go:399-402` shows a narrowed tier that holds no test `continue`s without running its command.
  - `TestLiveGate`'s "started no binary" row feeds a fake `Run`, which can only produce a line it was given. The absent-line state is reachable and unprescribed, so the gate could read it green.

- **LOW. Card 1 (`--copies`), D4 bound: the copies lease is unguarded on the inherited-slot path.**
  - `execution.runWithSlots` skips every acquire when `exe.inherited` is set, which happens when `LYM_GATE_SLOT` names a held slot (exploration of `internal/gatecli/run.go:149`).
  - Card 1 puts the n-times lease in `runWithSlots` but does not refuse or define `--copies` under an inherited slot.
  - A `--copies` call made inside a held slot would run n copies in one slot, against "one lease of n times the tier's own slot count".

- **LOW. Card 8 (`Reap` step 3): a hub whose reed server never started may never be pruned.**
  - Step 3 says it "prunes the hub's list entry once its server is stopped".
  - `StopServer` "reports false for no live server".
  - A hub that `new` made but no round ever brought up has no server, so if pruning waits on a true answer, its entry stays in `hubs.json` forever.
  - This state is reachable through `new` followed by `clear`. `TestReap` has no row for it.

- **LOW. Card 9 (`WrapRow`), D5 and D7: the review row's round is wrong when a review row resumes after its review file exists.**
  - The review row takes `ReviewRound+1` at start.
  - Suppose a crash or pause lands after the review file is written but before the row ends. On resume `ReviewRound` already returns n, so the start reap's strays and the models file go to round n+1.
  - The fix row then reads round n, so round n's stray report and models file are split from it. The state is reachable through crash recovery, and no test row covers it.

**Tool calls beyond the inherited context**
1. `internal/shedadapters/bouncer.go` around `settleUndecided`, `softCapRule` and `Call`: to confirm which round the Bouncer judges, against the live report's round. The context lacked how the round is resolved.
2. A grep of the `pairteardown` routes and the remove callers: to check that every teardown route passes through `RemovePair`. The context lacked how `Run` relates to `RemovePair`.
3. `TestRunTierNoTestNotice` in `internal/gatecli/tierrun_test.go`: to see how a tier that runs no test reports. The context lacked it.
4. `internal/gatecli/tierrun.go` 380-420 and 489-512: the empty `{test_targets}` branch, for the started-line guard.
5. A grep of the hub's `_bolt/_lym/config/gate.yaml` gates: to confirm the hub file holds only tuned keys, so `HubSetsGate` can answer false.
6. The same `gates` block plus a `configengine/doc.go` grep: to confirm template keys are filled in memory, not written into the hub file.
