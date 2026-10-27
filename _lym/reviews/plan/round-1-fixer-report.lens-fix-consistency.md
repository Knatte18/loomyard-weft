```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens. I checked whether the fixer's changes agree with the sibling cards, decisions and cross-references they touch.

**MEDIUM: card 13's `ReadCheck` no longer reports an absent record as absent when the directory is missing, and card 14 depends on it doing so.**
- **Changed text:** `_lym/plan/13-adopt-check-verb.md`, the `ReadCheck` sentence: "reads it through `state.ReadJSONNoMkdir`, so a read creates no directory, and reports false when absent".
- **Summary:** when `_board/.lym/adopt/` does not exist, the read fails with an error instead of reporting false. Card 14's `ensureCheck` is the twin that was left unchanged, and it can now fail on a hub's first `lym adopt`. Card 15 avoids this only because the fix adds a stat of `check.json` before it calls `ReadCheck`.
- **Evidence:**
  - `state.ReadJSONNoMkdir` takes `lock.AcquireReadLock(lockPath)` before it reads (`internal/state/state.go:175-181`).
  - `AcquireReadLock` is `flock.New(lockPath).RLock()` (`internal/lock/lock.go:70-75`). That opens the lock file for creation, which fails when its directory is absent. `ReadJSONNoMkdir` turns only a missing JSON file into `(zero, false, nil)`, not a missing directory.
  - Card 14 says "`ensureCheck` reads `check.json` in `dir`" and has the row "no record spawns once" (`_lym/plan/14-adopt-spawns-check.md:36,43`). Its test runs in a temp directory that exists, so it would not catch this.
  - Before this fix, `ReadJSON` ran `MkdirAll` first, so the absent case read as false.
- **Proposed fix:** either make `ReadCheck` stat the record and return false when it is absent (as card 15 does), or say in card 14 that `ensureCheck` handles an absent directory.

The other changes agree with their siblings:
- Card 14's "card 11's hub-base seed" matches card 11.
- Cards 7, 8 and 10 list `beamcli/doc.go` consistently.
- The stop-line suffix appears only in card 7.
- Removing the empty name in cards 2 and 3 still agrees with decision record line 137.
- The order change in card 10 matches its row list.
- Card 13's `TestHubCheckDeps` still gets `adopt`, through `hubreconcile` → `ReconcileHubWideAt`'s present-file branch.

Tool calls:
1. Grepped the plan cards and decision record for the stop line, empty-name, `ReadCheck`, `MigrateConfig` and `registerDobby` references, to find twins of the changed text across all cards, since my context held only some cards.
2. Read `lock.AcquireReadLock`, card 14's `ReadCheck` and `ensureCheck` lines, and decision record Decision 9. My context lacked `AcquireReadLock`'s behaviour on a missing directory and how card 14 uses `ReadCheck`.
