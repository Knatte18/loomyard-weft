---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/discussion/decision-record.md:130
    summary: D7 refuses an unregistered Master engine at wiring, which runs for every verb, contradicting D6's rule that no verb which spawns nothing is refused by a key
    fix: "In D7, replace the last sentence (\"A Master choice naming an engine the set does not hold is refused at wiring with the set's lookup error, as `Start` refuses it.\") with a rule that keeps wiring refusal-free for the engine name: wiring builds the instruction-file names lazily, and a Master choice whose engine the set does not hold is refused by the lookup error only where the names are first needed (the `MerriamBase` callers: `run`'s partition formation, `webstercli`'s `executionBatches` with no state, and `rebaseline`), with the way forward to set `master` in `webster.yaml` to an alias whose engine the set holds with `lyx config webster`, then re-run the verb. Add to D7 that `record-batch`, `reset`, `validate`, an attach or await, and loom `status` and `pause` call none of those and are never refused by the Master key. Add an acceptance criterion: with the `master` key naming an engine the set does not hold, `record-batch`, `reset`, `validate`, a `recover-batch` attach and loom `status` still run."
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:104
    summary: D6 places both spawn checks "before StartMaster" and "before recoverSpawn starts the strand", but each site has destructive or state-mutating steps ahead of the Start call, so a wrong-key refusal can already have changed the run
    fix: "In D6, pin each check ahead of the first mutation, not beside the Start call. For `websterengine.Run`, say the check is one of the entry gates beside the plan-approved and `deps.Batcher` checks, before `AcquireStateMutation`, because it reads only `deps.Roles` and the lookup: after that point `reclaimEntryTimeStrands` has stopped live strands, `--fresh` has run `archiveRunInPlace` and saved a new `State`, and the stale outcome and summary are archived. For recover-batch, say the check is the first step of the spawn branch, ahead of `recoverSpawn`'s `archiveStaleReport` and the `StopStrand` of the prior recovery strand, and ahead of `AcceptBatchFabricReference`. Add to the acceptance criteria that a refused `run` (also with `--fresh`) and a refused `recover-batch` leave `state.json`, the archived state, the prior report and any live strand unchanged."
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:103
    summary: D6 says the spawn check "sits at the spawn site" but `run` checks the recovery strand, which it never spawns, and the Bound's "default engine is the only one no longer checked there" has no clear referent
    fix: "In D6, reword the sentence \"The spawn check sits at the spawn site, not in the wiring, because ...\" to say the checks sit in the engine functions that spawn, not in the wiring, because the wiring runs once per invocation for every verb; and that `run`'s check of the recovery strand is a pre-flight for the later `recover-batch` spawns, which `run` itself never makes. In the Bound, replace \"the default engine is the only one no longer checked there\" with \"the wiring no longer checks any engine, so the default engine is checked only when a role's choice or an engine-less state entry resolves to it\"."
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:99
    summary: D6 reads a recovery strand's turn ends "through recoveryEngines", a map keyed by session id, but the turn-end read is keyed by the record's events path and names no session id
    fix: "In D6, after \"the recovery strand's sessions and turn ends through `recoveryEngines`\", add that a live recovery record's own session is the last entry of its `RecoverySessions`, and the turn-end read of that record (`TurnEndedAfter` over the record's `EventsPath`) resolves its engine from `recoveryEngines` under that entry; a record whose last entry has no `recoveryEngines` key is an engine-less entry under the three-way lookup."
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:118
    summary: The Bound says a wrong-default read of an engine-less entry "finds no forks or writes", but the one registered engine errors on a session whose parent transcript is missing, so the stated failure mode and its way forward are unsupported
    fix: "In D6's Bound, replace \"that read finds no forks or writes, so it can pass a run-exit audit or hide a run's own path from `reset`\" with the outcome per engine: a wrong-engine read ends in the engine's own audit error where that engine requires the session's transcript (the `claude` engine's `AuditForksIncremental` errors on a missing parent transcript), and in an empty audit, which can pass a run-exit audit or hide a run's own path from `reset`, where it does not. Give the audit-error case a way forward in the same sentence: set `engine:` in `shuttle.yaml` back to the engine that ran the session, then re-run the verb, or under the Webster row re-step the row. Keep the \"accepted while one engine is registered, revisited when `codex-engine` lands\" bound."
---

### [BLOCKING] D7's wiring refusal contradicts D6's no-key-check rule

**Location:** `_lyx/discussion/decision-record.md:130` against `:103` and `:107-108`.
**Issue:** D6 states that `webstercli`'s and loom's wiring run once per invocation for every verb, and builds its placement on that: `record-batch`, `reset`, `validate`, an attach, an await and loom `status` and `pause` check no key.
D7's last sentence refuses "a Master choice naming an engine the set does not hold ... at wiring".
Read as written, the wiring of every one of those verbs looks up Master's current choice and refuses, so a `master` key change can stop a verb that spawns nothing, and D6's statement that a key change cannot refuse an attach or await is false for that case.
The second D6 rule, that a session is read through its recorded engine, also loses its meaning if the current `master` key can refuse the read first.
The case is unreachable today while `modelspec` admits only registered engines, but the record is the plan writer's only input and the two Decisions give opposite instructions for the wiring.

### [MEDIUM] The spawn checks sit after the steps that change the run

**Location:** `_lyx/discussion/decision-record.md:104-105`.
**Issue:** The placement is stated as "before `StartMaster`" and "before `recoverSpawn` starts the strand".
In `websterengine.Run` (`internal/websterengine/runlevel.go`), everything between the state phase and `StartMaster` is already done by then: the entry-time reclaim stops live strands, `--fresh` calls `archiveRunInPlace` and saves a fresh `State`, and the stale outcome and summary are archived.
In `recoverSpawn` (`internal/websterengine/recoverbatch.go`), the stale report is archived and the prior recovery strand is stopped before `deps.Starter.Start`.
A check placed next to the Start call therefore refuses a wrong key after destroying the old run record or stopping a working strand, which the criterion "refused before any pane exists" does not rule out.
The check needs only the role's choice and the lookup, so it can sit ahead of all of them; the record should say so, since the placement is the point of the Decision.

### [LOW] "Spawn site" is not true of `run`'s recovery check

**Location:** `_lyx/discussion/decision-record.md:103` and `:117`.
**Issue:** The record says the check sits at the spawn site, then has `run` check the recovery strand, which only a later `recover-batch` invocation spawns.
The rule sentence and the Bound do state what `run` checks, so they are true of `run`; it is the framing sentence that overstates, and "the default engine is the only one no longer checked there" does not say where "there" is.
A plan writer can read either as a reason to move the recovery pre-flight out of `run`, which the acceptance criterion naming `recovery` before Master spawns forbids.

### [LOW] The turn-end read of a live recovery has no stated key

**Location:** `_lyx/discussion/decision-record.md:99`.
**Issue:** `recoveryEngines` maps session id to engine name, and D6 names the home record and the carry-forward for both batch writers, which settles the second-recovery and re-begin cases.
The turn-end read of an attached recovery (`TurnEndedAfter` in `internal/websterengine/recoverbatch.go`) takes the record's events path and an engine, not a session id.
D6 does not say which `recoveryEngines` entry serves it, so a second recovery could be read through the first recovery's entry or through the current default.

### [LOW] The wrong-default failure mode in the Bound is unsupported

**Location:** `_lyx/discussion/decision-record.md:118`.
**Issue:** The Bound says the read "finds no forks or writes".
`claudeengine`'s `AuditForksIncremental` returns an error when the parent transcript is missing and treats only a missing `subagents/` directory as zero forks, so with `claude` as the default and a session another engine ran, the read errors instead of passing.
Whether a second engine errors or returns an empty audit is its own contract and unknown here.
The error case also has no stated way forward, which `PATTERN-refusal-way-forward` asks of a webster refusal.
The open-risk bound itself (one engine registered, revisited at `codex-engine`) is stated and sufficient.

## Rejected

- The per-run state keys do not lack a home record: `State` holds `masterEngine`, `BatchState` holds `engine` and `recoveryEngines`, and the only two wholesale `BatchState` writers in non-test code are `beginbatch.go` and `recoverbatch.go`, both named for the carry-forward.
- The check site does not fire on `record-batch`, `reset`, `validate`, an attach, an await or loom `status` and `pause`: D6 places the checks in `Run` and in the recover-batch spawn branch, the only paths that start a strand, and `internal/webstercli/wiring.go` is the only place that currently refuses them.
- D6's premise that a shuttle run record does not outlive the run holds: `recoverbatch.go` removes a done recovery's `ShuttleRunDir`, and `shuttleengine` removes run directories on its own paths.
- A run record always carries a resolved engine name after `engine-registry` (`Start` records the name `engineFor` returns), so a post-change state entry is never engine-less; the engine-less case is pre-change state only.
