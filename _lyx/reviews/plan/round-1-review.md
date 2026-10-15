---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/01-spec-embeds-model-choice.md
    summary: Card 1 switches webster's Master and recovery specs to NewModelChoice(resolved), so the recovery spec carries Engine claude from fixtures that a shuttlefake.EngineSet refuses at Start; the fixture change that keeps those tests green lands only in card 7.
    fix: Add `internal/websterengine_test#newRunFixtureOver`, `internal/websterengine_test#newRecoverFixtureOver` and `internal/webstercli#newVerbsFixture` to card 1's Edit list, and say in card 1's Intent that these fixtures' role literals drop `Engine` claude to an empty engine, the default of their `shuttlefake.EngineSet`, in the same commit that makes the webster specs carry the resolved engine. Card 7 keeps these three targets for its own move to `shuttleengine.ModelChoice` role values. The combined overlay with this change and F2's passes `lyx loom validate-plan --overlay` with no new finding.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/10-merriam-instruction-files.md
    summary: Card 10 names run's partition formation and auto-rebaseline as triggers of the master engine not registered row and plans Run test rows for them, but card 9's roleAuditEngine entry gate refuses a master engine the set does not hold before Run reaches either MerriamBase call, so no real input reaches those triggers.
    fix: In card 10, drop `run` forming a partition or auto-rebaselining from the `master engine not registered` row's trigger, keeping the no-state verb, `validate --batches` and `rebaseline` triggers; drop the planned `TestRun_RefusesBeforeSpawn` and `TestRun_AutoRebaseline` rows and remove `internal/websterengine_test#TestRun_RefusesBeforeSpawn` and `internal/websterengine_test#TestRun_AutoRebaseline` from its Edit list; add one Intent sentence saying `Run` still calls the lister before each `MerriamBase` call but card 9's entry gate has already refused a `master` engine the set does not hold, so `Run` propagates the lister's error only as an internal error.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/11-seat-requires.md
    summary: A Discussion-Write advisor's refusal takes ModelKey{loom, discussion_advisors}, whose way forward is lyx config loom --set discussion_advisors=<alias>, but discussion_advisors is a list of {name, model} pairs, so that command overwrites the advisor list with a scalar instead of changing one advisor's model.
    fix: In card 11, give an advisor's refusal a way forward that names the advisor's own entry, for example `set the model of advisor <name> in discussion_advisors in loom.yaml to an alias whose engine meets them with lyx config loom, then re-run the verb, or under the role's row re-step the row`, either by adding the seat name to `ModelKey` or by a fourth `Wrap` form for a list-entry key; update the loom-section advisor row and the `TestEngine_Run_AdvisorStartFailureIsRecordedAndNamedToTheChair` and `TestRunner_Requires` rows to assert that text.
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/11-seat-requires.md
    summary: Card 11 removes the generic way forward from Runner.checkRequirementsAtStart, but lyx orch start's spec sets ForkSubagents true and its Start error is wrapped by no ModelKey, so an orch refusal on an engine without forks loses its only way forward, and the orch is out of this task's scope.
    fix: In card 11, keep a way forward on every `Start` requirements refusal no caller wraps; for example, `checkRequirementsAtStart` keeps today's generic clause and `ModelKey.Wrap` replaces it with the key-specific clause, which needs a typed form such as a `RequirementsError` holding the message without its way forward. Name that mechanism in the Intent, and add a `TestRunner_Requires` row asserting that an unwrapped refusal still carries the generic way forward and a wrapped one carries only the key-specific one.
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/plan/11-seat-requires.md
    summary: D8 requires each Requires row to be reached through a fake engine set whose engine lacks the requirement, but card 11 reaches the advisor row only through a fake shuttle returning a canned error, and names no test that reaches the shed-section MultiLLM row other than TestRunner_Requires' direct Wrap rows, which call only the error constructor.
    fix: In card 11, make the `TestEngine_Run_StartFailureEndingTheRunStopsEveryStartedSeat` table's two-seat row the MultiLLM case, with the zero `ModelKey`, asserting the zero key's seat way forward and no `AddStrand`; move the advisor row from `TestEngine_Run_AdvisorStartFailureIsRecordedAndNamedToTheChair`'s fake shuttle onto the same real `Runner` over `shuttlefake.EngineSet(&shuttlefake.Engine{})`, starting a chair whose spec needs nothing it lacks, or else state in the Intent why the advisor row cannot be reached that way.
---

### [BLOCKING] Card 1 breaks webster's recover tests until card 7

**Location:** `_lyx/plan/01-spec-embeds-model-choice.md`

**Issue:** Card 1's Intent has `webster`'s Master spec in `Run` and recovery spec in `recoverSpawn` build their choice with `shuttleengine.NewModelChoice(resolved)`, so they carry the alias's engine.
The fixtures feeding those specs set `{Engine: "claude", ...}` role literals: `internal/websterengine/recoverbatch_test.go:97`, `internal/websterengine/runlevel_test.go:210` and `internal/webstercli/verbs_test.go:127`.
`newRecoverFixtureOver` and `newVerbsFixture` hand `recoverSpawn` a real `shuttleengine.Runner` over `shuttlefake.EngineSet(engine)` as `Starter`.
That set registers only `fake` (`internal/testkit/shuttlefake/shuttlefake.go:68-72`), and `Runner.start` resolves `spec.Engine` through `engineFor` before anything else (`internal/shuttleengine/run.go:491`), so a recovery spec naming `claude` is refused with `ErrEngineNotRegistered`.
After card 1's commit, every recovery spawn in those fixtures fails, including `TestRecoverSpawnOrAttach` and webstercli's recover-batch tests.
Card 7 moves the fixtures to an empty engine ("since a role naming `claude` would now be refused by a set that registers only `fake`"), but card 1 is the card that makes that refusal happen, and it lists none of the three fixtures.
Card 1 therefore depends on a change that only card 7 makes.

### [MEDIUM] Run's MerriamBase refusal triggers are unreachable behind card 9's entry gate

**Location:** `_lyx/plan/10-merriam-instruction-files.md`

**Issue:** Card 9 puts `roleAuditEngine` among `Run`'s entry gates, right after the plan-approved and `deps.Batcher` checks (`internal/websterengine/runlevel.go:397-400`).
It refuses a `master` choice "on an engine without fork audit or one the engine set does not hold".
Both of `Run`'s `MerriamBase` calls come later, at partition formation (`runlevel.go:464`) and auto-rebaseline (`runlevel.go:515`).
A `master` engine the set does not hold is the only input that makes `MasterInstructionFiles` fail, and card 9's gate rejects it first.
Card 10 still lists `run` forming a partition or auto-rebaselining as triggers of the new `master engine not registered` row.
Its `TestRun_RefusesBeforeSpawn` and `TestRun_AutoRebaseline` rows reach those triggers only by injecting a failing lister beside roles that pass card 9's gate, which is not an input production can produce.
The upstream check that rejects the named input is card 9's `roleAuditEngine` entry gate.
The webstercli triggers (`executionBatches` with no state, `validate --batches`, `rebaseline`) stay reachable, since `executionBatches` is never called on `run`'s path.

### [MEDIUM] The advisor way forward sets a list key to a scalar alias

**Location:** `_lyx/plan/11-seat-requires.md`

**Issue:** Card 11 gives each Discussion-Write advisor `ModelKey{loom, discussion_advisors}`, and `Wrap` turns a named key into `set <Key> in <Module>.yaml to an alias ... with lyx config <Module> --set <Key>=<alias>`.
`discussion_advisors` is a list of `{name, model}` pairs (`internal/loomengine/template.yaml:4`, `internal/loomengine/config.go:17`), not a model spec.
Following the way forward replaces the whole advisor list with one string instead of changing the refused advisor's model, so the refusal names a recovery step that does not work.
The other keys card 11 names (`discussion`, `plan`, `fan_review`) and card 12's (`master`, `driver`) are scalar model specs, where the form fits.

### [MEDIUM] Dropping shuttle's generic way forward strips it from the orch's refusal

**Location:** `_lyx/plan/11-seat-requires.md`

**Issue:** Card 11 makes `checkRequirementsAtStart` stop appending `; way forward: give the role a model alias of an engine that meets them` (`internal/shuttleengine/requires.go:106`), leaving the way forward to the caller that resolved the key.
`ForkSubagents` implies `forks`, and `lyx orch start` builds its spec with `ForkSubagents: true` (`internal/orchcli/start.go:74`), then calls `s.runner.Start(spec)` (`start.go:55`) with no `ModelKey` wrap.
On a default engine without forks, the orch's `Start` refusal would carry no way forward at all.
The decision record puts the orch's model and spec out of scope, so card 11 cannot fix this by wrapping at `orchcli`.
The Intent's "So no requirements refusal leaves a seat without a way forward" covers seats only.

### [MEDIUM] Two Requires rows are not reached through a fake engine set

**Location:** `_lyx/plan/11-seat-requires.md`

**Issue:** D8 says "A table-driven test reaches each row through a fake engine set whose engine does not support the requirement."
Card 11 does this for the `discussion`, `plan` and `fan_review` chair rows in `TestEngine_Run_StartFailureEndingTheRunStopsEveryStartedSeat`.
The `discussion_advisors` row is reached in `TestEngine_Run_AdvisorStartFailureIsRecordedAndNamedToTheChair` through a fake shuttle that returns "the error a real `Runner`'s `Start` returns", so no engine set is involved and shuttle's refusal is never run.
The shed-section MultiLLM row (zero key) has no named test that drives a seat to it.
`TestRunner_Requires`' `ModelKey.Wrap` rows call only the error wrapper, which the rubric says does not reach the refusal.
The `StartFailure` table's "two-seat table's chair with `peer-messaging`" row does not name its key, so it is not stated to be the zero-key case.
`PATTERN-refusal-way-forward` requires that a test reach each new row.
