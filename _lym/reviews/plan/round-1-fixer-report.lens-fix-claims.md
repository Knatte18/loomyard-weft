```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: I checked every sentence the fix added or rewrote that states a fact about the code against HEAD.

**[MEDIUM] Card 21, `_lym/plan/21-decision-notice-weaver.md` Intent: "`notifyLiveRuns` computes `weaverActive` once … and passes it down, to `heldBackReason` and, through `addedNoticeLines`, to `decisionNoticeInline`."**
`notifyLiveRuns` does not call `addedNoticeLines`, so it cannot pass `weaverActive` to it.
- Evidence: the decision verb calls `addedNoticeLines` inside the `linesFor` closure it hands to `notifyLiveRuns` (`internal/loomcli/decision.go:1030-1032`: `notifyLiveRuns(deps, halted, func(strand string) []string { return addedNoticeLines(deps, added, strand) })`).
- `notifyLiveRuns`'s own signature is `notifyLiveRuns(deps, halted, linesFor func(strand string) []string)` (`decision.go:469`), and nothing in it reaches `addedNoticeLines`.
- For the claim to hold, the change must do one of two things. Either `linesFor` takes `weaverActive` as a parameter, or the verb that builds the closure computes `weaverActive` itself, and then that verb belongs in the card's Edit list.
- The card names neither change.

Every other claim I checked holds:
- **Card 1:** `ConflictRecordName`'s comment says "in the scratch directory" (`internal/runplan/conflict.go:122`). The `Geometry.ScratchDir` field comment lists the plan-conflict record (`internal/websterengine/geometry.go:44`).
- **Card 2:** `configengine.CategoryGuard` exists (`internal/configengine/layers.go:64`). `LocalDenied` is derived from the Spec (`internal/configreg/configreg.go:151`).
- **Card 11:** the tripwire fails on a listed site it does not find (`cmd/lym/verifiedtree_test.go:81-84`).
- **Card 12:** `gatefixer.Record` has `Status` and `DeadReason` (`internal/gatefixer/record.go:43-46`). `Await` never marks the record terminal (`internal/gatefixer/await.go:40`). `SpawnOrAttach` returns a live record without spawning (`internal/gatefixer/spawn.go:93-94`).
- **Card 22:**
  - `websterengine.BatchState` is an alias of `runplan.BatchState` (`internal/websterengine/runplan.go:21`), so moving `lastCardSHA` to `*runplan.BatchState` is sound.
  - With a nil `Setting`, `Active` answers `webster` when neither implementer has begun, per card 6.
  - `dest` is `archiveDir/webster` (`internal/loomshed/planwrite.go:150`), so the parent-directory join is right.
  - The slot const block exists (`internal/loomshed/prrework.go:24`).

Tool calls:
1. Grepped configengine and configreg for `CategoryGuard` and `LocalDenied`, gatefixer for `Status` and `DeadReason`, websterengine for `BatchState`, card 6 for `Setting`, and `decision.go:1025-1032`. My context had none of these definitions or the caller of `addedNoticeLines`.
2. Grepped for gatefixer's `Record` struct and websterengine's `BatchState` alias, because the first grep matched `Result`'s fields and not `Record`'s.
3. Grepped `gatefixer/record.go` for `Record`'s `Status` and `DeadReason`, because call 2 did not match the struct.
