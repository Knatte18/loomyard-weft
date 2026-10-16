---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:94-96
    summary: "D6 does not say the new engine state carries forward when begin-batch re-begins or recover-batch respawns a batch, so earlier recovery sessions lose their engine and read as the current default."
    fix: "In D6, name where the keys live and add the carry-forward. State that `recoveryEngines` and `engine` are fields of the batch record (`BatchState`) beside `RecoverySessions` and `SessionID`. State that begin-batch's re-begin record and recover-batch's respawn record both carry the prior batch's `recoveryEngines` forward, the way they already carry `RecoverySessions`, so an entry is dropped only when its session id is. Add an acceptance criterion: after two recoveries of one batch, or a re-begin after a recovery, `reset` and `recover-batch` read the first recovery's session through its own recorded engine."
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:99-103
    summary: "D6 puts the per-verb engine check in each verb's wiring, but wiring runs once per invocation for every verb and recover-batch also attaches and awaits, so the placement is wrong or unstated for recover-batch, run and loom."
    fix: "Replace `Each verb's wiring checks only the role engines that verb spawns` with a placement that is true of the code. Put the check at the spawn site, not in the wiring prologue. `websterengine.Run` makes it before `StartMaster`, and recover-batch's spawn branch makes it before `recoverSpawn` starts the strand. Both go through the lookup the wiring hands in, applied to the role's `ModelChoice` engine, so `webstercli` and `loomcli` share one check and `websterengine` still resolves no engine set itself. State that recover-batch's attach, await and terminal paths, and every verb that spawns nothing, check no key. State that loom's Webster row carries the `run` check and that `lyx loom status` and `pause` do not (loom's wiring also runs for them). Then re-verify the two sentences that depend on the placement: the lookup refusal is the only refusal an attach or await can hit, and the criterion at line 169 refuses only the spawn."
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:100-101
    summary: "D6 says each verb checks only the engines it spawns, yet run also checks the recovery strand's engine, which only Master's later recover-batch calls spawn; the Bound sentence repeats the claim."
    fix: "Reword the rule sentence and the `Bound:` sentence so they say what is true. For example, `run` checks the engine of every strand the run can spawn, Master's and, through Master's recover-batch, the recovery strand's, so a wrong key fails before Master spends a turn; `recover-batch` checks the recovery strand's again because its key can change mid-run. Keep acceptance criteria 167-169 as they are."
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:106-110
    summary: "A state entry with no engine reads as the current default, and the bound covers only a default without fork audit; if the default was changed to another engine that audits forks, the old session is read through the wrong provider with no refusal."
    fix: "In D6, decide and state the bound for an engine-less entry. Either (a) add to the `Bound:` paragraph that an engine-less entry is read as the current default and is wrong when `engine:` in `shuttle.yaml` differs from the engine that ran it, and that a wrong read finds no forks or writes and so can pass an audit or hide an own-path; and add that as an Open risk, accepted while one engine is registered and revisited when `codex-engine` lands; or (b) name a mechanism that stamps the engine onto a pre-change entry the first time a post-change verb loads the state, and say which engine it stamps. Pick one in the Decision, and add a clause saying what happens when a recorded engine name is not held by the engine set (a binary change), since the `internal error` sentence covers only a name without fork audit."
---

### [BLOCKING] Engine state is not carried across re-begin and respawn

**Location:** `_lyx/discussion/decision-record.md:94-96`, checked against `internal/websterengine/beginbatch.go:660-695` and `internal/websterengine/recoverbatch.go:345-385`.

**Issue:** D6 records `recoveryEngines` as "a map from session id to engine name beside `RecoverySessions`".
`RecoverySessions` lives on `BatchState`, and both writers rebuild that struct wholesale.
begin-batch copies `prior.RecoverySessions` into its fresh record, and `recoverSpawn` clones it and appends the new session id.
Neither is told to do the same for the engine map, nor for a batch's `engine`.
A plan writer who adds only the spawn-time write loses the first recovery's engine when the second recovery or a re-begin replaces the record.
`loadRunWrites` still audits that session id, finds no entry, and falls through to the legacy rule, which reads it as the current default.
That is the silent read the round's first focus question asks to rule out, and acceptance criterion 172 cannot hold after a second recovery.
The state keys' home (the batch record, not `State`) is also only implied by "beside".

### [BLOCKING] The per-verb engine check has no true placement

**Location:** `_lyx/discussion/decision-record.md:99-103`, checked against `internal/webstercli/wiring.go` (`wire`, `wireHub`, `wireStandalone`), `internal/webstercli/recoverbatch.go` and `internal/loomcli/wiring.go:520-560`.

**Issue:** D6 says "Each verb's wiring checks only the role engines that verb spawns".
`webstercli`'s wiring runs once in the pre-run for every verb and cannot tell which verb is running, and `loomcli`'s `wire` runs for `status` and `pause` too, as its own comment says.
A check in either place cannot be per-verb.
In `webstercli` it would refuse `record-batch`, `reset` and `validate`, which D6 says check no key.
Even for `recover-batch` it fires on an attach to a live recovery, an await and a terminal re-read, none of which spawn.
The spawn-or-attach decision is made later, inside `RecoverSpawnOrAttach`.
A mid-run key change would then refuse an attach to a recovery that is running on a fork-audit engine, contradicting the sentence that a key change does not change how a session is read.
For loom, `run` is the Webster row and no verb, so "run checks" has no site there.
Round 2's F2 was recorded as fixed, but the fix names verbs without placing the check.
The suggested placement in `fix` keeps `PATTERN-told-geometry`, since `websterengine` uses the lookup it is handed.

### [MEDIUM] `run` is said to check only what it spawns but checks the recovery strand too

**Location:** `_lyx/discussion/decision-record.md:100-101` and the `Bound:` paragraph at line 110.

**Issue:** D6 gives the rule "each verb's wiring checks only the role engines that verb spawns, since every session of both roles is fork-audited".
It then has `run` check the recovery strand's engine, which `run` never spawns; Master's `recover-batch` calls do.
Round 1's F1 asked for the early recovery check, and it is worth keeping, but the rule and the `Bound:` sentence ("each spawning verb checks the role engines it spawns on") are false for `run` as written.
A reader who follows the rule drops the recovery check from `run` and fails criterion 168.

### [MEDIUM] The legacy engine-less read is bounded only for a default without fork audit

**Location:** `_lyx/discussion/decision-record.md:106-110`.

**Issue:** An entry with no engine name "reads as the current default", and the lookup refuses it only when that default has no fork audit.
If `engine:` in `shuttle.yaml` was changed to another engine that does audit forks, the pre-change session is parsed through the wrong provider's records.
The attack-surface question applies: this escape hatch lets a read through, and the stated bound does not cover that case.
The audit then finds no forks or writes; a run-exit audit can pass on an empty result, and a `reset` can class the run's own paths as foreign.
This cannot happen while one engine is registered, so accepting it may be right, but the Decision has to say so and name when it stops being accepted.
The sentence that calls a recorded name without fork audit "an internal error" also leaves out the neighbouring case of a recorded name the engine set does not hold, which a binary change reaches and which has no stated outcome.

## Rejected

- Webster state keys lacking a way forward or an acceptance criterion: the lookup refusal has both (line 107 and criterion 170), and each wiring refusal names its key (criteria 167-169).
- The second focus question's `reset` and `record-batch` placement: D6 correctly says they spawn nothing, and the lookup refusal is the only one they can hit.
- Where `masterEngine` is written: D6 writes it beside `MasterSessionID` at Master's spawn, and `runlevel.go` saves `MasterSessionID` in a second save after `FindRun`, so the plan writer has one obvious site; no finding.
