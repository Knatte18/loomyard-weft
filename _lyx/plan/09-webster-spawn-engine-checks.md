# Card 9 — Webster checks each role's engine before it spawns

**Create:**
- `plan:internal/websterengine#roleAuditEngine` -> `func roleAuditEngine(engines AuditEngines, role Role, choice shuttleengine.ModelChoice) error`

**Edit:**
- `internal/websterengine#Run`
- `internal/websterengine#RecoverSpawnOrAttach`
- `internal/websterengine/doc.go`
- `internal/websterengine_test#TestRun_RefusesBeforeSpawn`
- `internal/websterengine_test#TestRun_MasterSpawn`
- `internal/websterengine_test#TestRecoverSpawnOrAttach`
- `contracts/specs/refusal-spec.md`

**Intent:**
D6's spawn side.
`roleAuditEngine` applies the told lookup to the role's choice engine, an empty engine being the default: it returns nil when the engine resolves and has fork audit (`AuditEngine`), and otherwise a refusal naming the role's key, `master` or `recovery` in `webster.yaml`, and the engine, wrapping the lookup's error for an engine the set does not hold, with the way forward `set <key> in webster.yaml to an alias whose engine audits forks with lyx config webster, then re-run the verb, or under the Webster row re-step the row`.
`webstercli` and loom's Webster row reach it through the same `Engines` they wire, so both share one check.

`Run` checks both roles, `RoleMaster` and `RoleRecovery`, as one of its entry gates right after the plan-approved and `deps.Batcher` checks and before `AcquireStateMutation`, reading only `deps.Roles` and `deps.Engines`: no live strand has been stopped, no `--fresh` archive or new `State` saved and no stale outcome or summary archived when it refuses.
Its recovery check is a pre-flight for the later `recover-batch` spawns, which `run` never makes itself; a missing role entry keeps today's refusal.
`RecoverSpawnOrAttach` checks the recovery role again as the first step of its spawn branch, right after its return to a live, non-terminal recovery record and ahead of the exhausted-recoveries refusal, `AcceptBatchFabricReference`, the contract-file check and `recoverSpawn`'s `archiveStaleReport` and `StopStrand`, because the `recovery` key can change mid-run.
The attach, await and terminal paths, and every verb that spawns nothing, check no key.
Bound: the default engine is checked only when a role's choice resolves to it, so a default without fork audit is refused only then; the check skips no audit, since every later read still goes through `sessionAuditEngine`.
`doc.go` states where the checks sit.

`contracts/specs/refusal-spec.md`'s webster section gains the row `role engine without fork audit`: `run` finds `master`'s or `recovery`'s choice on an engine without fork audit or one the engine set does not hold, or `recover-batch`'s spawn finds `recovery`'s, correctness halt, with the way forward above.

Tests:
- `TestRun_RefusesBeforeSpawn` gains rows over an engine set whose engine `x` hides its fork audit (`struct{ shuttleengine.Engine }{fake}`): Master's choice on `x` is refused naming `master`, the recovery choice on `x` naming `recovery`, a choice on an engine the set does not hold naming its key, and a master choice with an empty engine while the set's default is `x` naming `master`, so the default is refused when a role's choice resolves to it; each row, run plainly and with `--fresh` over a recorded state, a recorded Master strand and a stale outcome, asserts no `StartMaster` call, no `StopStrand` call, `state.json`'s bytes unchanged, no archive written and the outcome left in place.
- `TestRun_MasterSpawn` gains a row whose set's default hides its fork audit while both roles name an engine `a` that audits: Master spawns.
- `TestRecoverSpawnOrAttach` gains rows: a recovery choice on `x`, and one on an engine the set does not hold, each with a terminal prior record, is refused naming `recovery` with no `Start`, the prior report still at its path, the prior strand not stopped and `state.json`'s bytes unchanged; the choice on `x` with a live, non-terminal recovery record attaches to it, not refused.

**ImpactSummary:** two entry checks in webster's two spawning functions and one new refusal row; no other verb changes.

**Commit:** 9: webster checks each role's engine before it spawns
