# Round 3 fixer report — discussion

## Changes

- F1 — fixed as proposed.
  D6 places `masterEngine` on `State` and `engine` and `recoveryEngines` on `BatchState`, beside `SessionID` and `RecoverySessions`.
  begin-batch's re-begin record and recover-batch's respawn record carry `recoveryEngines` forward as they carry `RecoverySessions`, and a batch's `engine` follows its `SessionID`, since the respawn record (`internal/websterengine/recoverbatch.go`) does not carry `SessionID`.
  New acceptance criterion: after two recoveries, or a re-begin after a recovery, `reset` and `recover-batch` read the first recovery's session through its own recorded engine.
- F2 — fixed as proposed.
  The check now sits at the spawn sites: `websterengine.Run` before `StartMaster`, and recover-batch's spawn branch before `recoverSpawn`, both through the lookup the wiring hands in.
  recover-batch's attach, await and terminal paths, verbs that spawn nothing, and `lyx loom status` and `pause` check no key; loom's Webster row carries `run`'s check.
  The lookup-refusal sentence now says those are the only refusals an attach, an await or a non-spawning verb can hit.
  Added a criterion that an attach or await after a mid-run `recovery` key change is not refused, which pins the placement.
- F3 — fixed as proposed.
  The rule sentence says `run` checks every strand the run can spawn, including the recovery strand through Master's `recover-batch` calls, and the `Bound:` sentence names `run`'s and recover-batch's checks.
  Criteria 167-169 are unchanged.
- F4 — fixed with option (a).
  The `Bound:` paragraph states that an engine-less entry is wrong when `engine:` differs from the engine that ran it, and that the wrong read can pass an audit or hide an own path.
  This is added as an Open risk, accepted while one engine is registered and revisited at `codex-engine`.
  The lookup now names the binary-change case: a recorded engine name the engine set does not hold is refused naming that engine, with a way forward and an acceptance criterion.
- `support-log.md` gains a Round 3 entry.

## Disputed

None.

## Deferred

None.
