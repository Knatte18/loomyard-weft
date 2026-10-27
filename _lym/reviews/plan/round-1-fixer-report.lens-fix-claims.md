```yaml
lens: fix-claims
outcome: reported
```

Task: check the fix-claims lens, meaning whether each sentence the fix added or rewrote holds against the code at HEAD.

no findings

Claims I verified:
- Card 8 and card 13 rely on `ReadJSONNoMkdir` creating no directory. It holds: `lock.AcquireReadLock` calls `flock.New(lockPath).RLock()` with no MkdirAll. With the directory missing, the read fails, which card 8 handles as "the worktree gone".
- Card 11 says `MigrateHubBase` leaves the fingerprint unchanged. It holds: `fingerprintOf` hashes only name, flags, open maps, per-name maps, template and overlay, and no func field.
- Card 11 says `lym fabric reconcile` (card 14) runs the hub-base seed. It holds: `internal/fabriccli/fabric.go:748` calls `ReconcileHubWideAt`.
- Card 11 says a `landing` list `[0, 1]` migrates to `finalize {tiers: [0, 1], scope: all}`. It holds: `retiredGateLists["landing"]` maps to `{"finalize", scopeAll}` at `internal/gateslot/config.go:398`.
- Card 13 says a tier that refuses after `Started` returns without `Ended`. It holds: `report` returns on `result.refusal` before writing the record (`internal/gatecli/run.go:448-451`), and card 12 calls `Ended` only from what `report` writes.
- Card 10 says the task-pair and loose rows of `TestDobbyCmd_RefusesAPathItCannotPlace` have no `.lym/beam` directory. It holds: hubforge creates no beam directory, and `placeDobby` refuses before `Run` and before any append.
- Card 7's kept sentence that the refusal of a path it cannot place comes before anything is registered or locked still holds, because `placeDobby` runs before `Run`.

Tool calls:
1. Read `lock.AcquireReadLock` and `configreg.fingerprintOf`: whether a read lock creates a directory and what the fingerprint hashes.
2. Read `gatecli/run.go` `report`, card 12's observer lines and `fabriccli/fabric.go:735-755`: whether a refusal skips `Ended` and whether fabric reconcile reaches `ReconcileHubWideAt`.
3. Grepped `lock.go` and `hubforge` for directory creation: whether the lock or hubforge creates `.lym/beam`.
4. Read the body of `AcquireReadLock`: the earlier grep did not show its body.
