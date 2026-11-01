```yaml
lens: fix-affected-set
outcome: reported
```

Fork task: rerun each set-premise finding's search and check the fix covers every member.

Unchecked list: none. I reran F1, F2, F4, F8 and F9 with searches. F3's and F5–F7's premises are not sets.

**[LOW] Card 4 (`_lym/plan/04-await-resolved-cap.md`), the rewritten Intent sentences on `await.go`'s header and `awaitVerb`'s comment**
- **Summary:** F8 fixed only `await.go`, but two comments in `internal/shuttlecli/cli.go` make the same claim (that await reads no config), and no card targets `cli.go`.
- **Evidence:**
  - `internal/shuttlecli/cli.go:9`: "await is the verb that skips the resolution, so it runs the same with no hub, git repository, config or engine."
  - `internal/shuttlecli/cli.go:43`: "The await verb skips it too, since it stats told paths and needs no geometry, config or engine."
  - After card 4, the verb loads claude.yaml and shuttle.yaml through `shuttleengines.KeepWarm` inside a worktree.
  - `grep shuttlecli _lym/plan/[0-9]*.md` finds card 4's targets only, so `cli.go` is in no card.

Members checked and covered:
- **F2:** `helperSlotWaitEnv` uses (`test_integration_test.go:64-65`, `:113`, `:123`, `:902`, `:1008`, `:1038`) are covered by card 11's Delete, `runGateHelper` and `TestGateTest_Scenario` targets. `newCommand`'s `slotWait` comment (`cli.go:39`) is a card 11 Edit target. `cli_test.go:62` is now targeted.
- **F4:** `docs/overview.md:494` is edited by card 11. `webster-body-implementer.md:100` is rewritten by card 15. `loom-plan-spec.md:320` and `:595`, and `plancontext.go:147`, are now in card 14. `plancontext.go:29` and `:121` state no reason and stay true.
- **F1:** the other run files have one writer each: `phase.json`, `exit.json` and `line.json` by the runner, `process.json` by an exclusive create, `delivered` by a create. Card 12 writes no new record.
- **F9:** the remaining Extends groups on cards 1, 2, 3, 10, 11 and 14 were not part of F9's premise.

Tool calls:
1. Searched tests for `for up to`, `DefaultAwaitCap`, `slotWait` and `helperSlotWaitEnv` (F2's set); my context held only `cli_test.go`.
2. Searched specs, patterns, docs and code for the old `--wait` rationale (F4's set); my context held only the review's three sites.
3. Searched the shuttlecli comments on await, card 12's records and every card's Extends docs (F8, F1, F9); I had not read `shuttlecli/cli.go` or card 12.
4. Checked which cards target shuttlecli or the implementer stencil, to confirm whether `cli.go` and the stencil are covered.
