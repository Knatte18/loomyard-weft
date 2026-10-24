```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: checking that every claim the fix added or rewrote holds against the code at HEAD.

no findings

Claims checked and found to hold:
- **Card 9:** at that card, an upsert carrying `order` or `beam` is refused by the key allowlist before `normalizeTask` runs. `Store.UpsertTask` calls `validateUpsertFields` first (`internal/boardengine/store.go:485`). The replay calls `normalizeTask` itself (`replay.go:283`, `:365`).
- **Card 19:** the pre-run wrap "runs from the hub's prime worktree only" is inside `Command`'s `PersistentPreRunE` (`internal/orchcli/cli.go:110`, `:122`, `:145`), and `Command` is on card 19's Edit list. `refusalCase` runs for both the task worktree and the records prime, so the fixer was right not to have it assert the beam verbs.
- **Cards 2 and 8:** the origin record is a committed file in the records worktree (`OriginRecordPathFor`, `origin.go:78`, plus the origin-commit subject prefix at `origin.go:48`). So `git checkout HEAD -- <record path>` restores a deleted record, and `lyx fabric reconcile` recreates a missing records worktree (the `Reconcile` doc).
- **Card 24:** pair teardown ends the pair's session by slug (`internal/pairteardown/cancel.go:47-50`, `:117`), so the beam's session ends at the beam's teardown. `OrchLive` reads no strand where no orch state records one (card 19's rows), and card 21's `Orchs` takes only worktrees that record a strand. The `--window` flag help is inside batten's `Command` (`internal/battencli/cli.go:277`, `:337`).
- **Card 35:** `startGuard.refuse`'s left-pair check is gated on `!found` (`startguard.go:78`), so the stated contrast with the `Promoted` check holds.
- **Card 23:** today `stepRecordsPrimeRefusal` asserts "records sibling" and `stepNonPrimeRefusal` asserts both the task slug and the prime name. The "still names" sentences match what those steps assert.

Tool calls:
1. Read `internal/orchcli/cli.go`'s function boundaries and `origin.go`'s `ReadOriginFor`, to place the orch pre-run wrap and see how a missing record is detected; my context did not show either.
2. Grepped `fabricengine` for whether the origin record is committed, to back the `git checkout HEAD --` claim.
3. Grepped `internal/pairteardown` for which reed session teardown ends, to back card 24's claim about what ends the session.
4. Grepped `task.go` for where `normalizeTask` is called, to check card 9's ordering claim.
5. Read `store.go`'s `validateUpsertFields` and the order of checks in `UpsertTask`, to confirm the allowlist runs before `normalizeTask`.
