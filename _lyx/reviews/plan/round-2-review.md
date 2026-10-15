---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/08-webster-session-engines.md:12
    summary: Card 8 renames RunDeps.Engine to Engines but leaves shedrecipe.websterEntry, its tests and battencli intentionallyNil on the old field, so the tree stops compiling at card 8
    fix: >-
      In 08-webster-session-engines.md add these targets to the Edit group:
      `internal/shedrecipe#websterEntry`, `internal/shedrecipe#zeroEnvField`, `internal/shedrecipe#simpleEntryCases`,
      `internal/shedrecipe#TestWebsterEntry_SeamFields` and `internal/battencli#intentionallyNil`.
      Add one Intent sentence: `websterEntry` validates `Env.WebsterDeps.Engines` in place of `Engine`, naming `WebsterDeps.Engines` in its refusal,
      and the shedrecipe test field lists and battencli's `intentionallyNil` key move from `WebsterDeps.Engine` to `WebsterDeps.Engines`.
      This fix was checked with `lyx loom validate-plan --overlay` (combined with F2 and F3) and adds no blocking finding over the baseline.
  - id: F2
    severity: BLOCKING
    class: scope
    location: _lyx/plan/10-merriam-instruction-files.md:6
    summary: Card 10 adds the nil-able func field RunDeps.InstructionFiles but not envkit's nilLegal entry, so NilSeams reports it as an unwired seam and the envkit and battencli seam tests fail
    fix: >-
      In 10-merriam-instruction-files.md add `internal/testkit/envkit#nilLegal` to the Edit group and one Intent sentence:
      a nil `RunDeps.InstructionFiles` means none, so `nilLegal` gains `WebsterDeps.InstructionFiles`,
      which keeps `NilSeams(FullEnv)` empty and leaves battencli's `intentionallyNil` list unchanged.
      Checked with `lyx loom validate-plan --overlay` together with F1 and F3.
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/10-merriam-instruction-files.md:29
    summary: Card 10 never fills RunDeps.InstructionFiles in websterCLI.runDeps, so lyx webster run forms its partition counting no instruction file at all
    fix: >-
      In 10-merriam-instruction-files.md add `internal/webstercli#websterCLI.runDeps`, `internal/webstercli#TestWire_StandaloneMode` and `internal/webstercli#TestWire_ModeHubSelectsHubMode` to the Edit group.
      Add to the Intent that `websterCLI.runDeps` copies the CLI's lister into `RunDeps.InstructionFiles`, as `loomCLI.wire` does,
      and that both wiring tests assert `runDeps().InstructionFiles` is non-nil after wiring, so the `run` verb counts the files Master's engine names.
      Checked with `lyx loom validate-plan --overlay` together with F1 and F2.
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/11-seat-requires.md:39
    summary: The generic Start way forward kept for unwrapped callers tells lyx orch start to give the role a model alias, but the orch takes a raw model string and its engine is shuttle.yaml's default
    fix: >-
      In 11-seat-requires.md change the Intent so the generic clause `startRequirementsError.Error` appends is
      `give the role a model alias of an engine that meets them, or for a role that takes no alias set engine: in shuttle.yaml to one that does, then re-run the verb`,
      and name `lyx orch start` as the caller it serves (orchSpec sets ForkSubagents, passes orch.yaml's model string verbatim and sets no Engine).
      Make the `TestRunner_Requires` assertion on the unwrapped refusal check both halves of that clause.
      No target changes.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/10-merriam-instruction-files.md:39
    summary: The master engine not registered row says a verb run with no state reaches it, which includes run (refused earlier by card 9's gate), and it leaves the two webstercli callers free to append the transient clause beside the master way forward
    fix: >-
      In 10-merriam-instruction-files.md reword the row trigger to name the verbs that reach it: `validate` and `await-batch` run with no state, `validate --batches` and `rebaseline`;
      say `run` is refused earlier by the `role engine without fork audit` row.
      State in the Intent that `executionBatches` and `rebaselineCmd` return the lister's error without the `way forward: transient, re-run the verb` clause they append to a `MerriamBase` failure.
      Make the `TestValidateCmd_BatchesFlag` and `TestRebaselineCmd_Refusals` rows assert the message holds the `set master in webster.yaml` way forward and no `transient`.
      No target changes.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/10-merriam-instruction-files.md:39
    summary: Card 10 leaves the existing refusal-spec row Merriam start base unreadable naming CLAUDE.md and CLAUDE.local.md, which MerriamBase no longer names
    fix: >-
      In 10-merriam-instruction-files.md add to the Intent that the existing `Merriam start base unreadable` row in `contracts/specs/refusal-spec.md` is reworded from a present `CLAUDE.md` or `CLAUDE.local.md` to a present instruction file Master's engine names.
      `contracts/specs/refusal-spec.md` is already a target of the card.
  - id: F7
    severity: LOW
    class: scope
    location: _lyx/plan/08-webster-session-engines.md:77
    summary: Card 8's Intent states no bound for an engine-less session entry read through a default engine that audits forks, which the decision record says can pass an audit or hide a run's own path
    fix: >-
      In 08-webster-session-engines.md add after the third `sessionAuditEngine` bullet one sentence:
      an engine-less entry read through a default that differs from the engine that ran the session either ends in that engine's own audit error or returns an empty audit, which can pass a run-exit audit or hide a run's own path from `reset`;
      the bound is the single registered engine, and the `engine:` value in shuttle.yaml is the way forward for the first case.
      No target changes.
  - id: F8
    severity: LOW
    class: design
    location: _lyx/plan/11-seat-requires.md:3
    summary: ModelKey, ModelKey.Wrap and startRequirementsError are a public shuttleengine surface used by six packages across cards 11 and 12, yet they ride in the seat-requires card instead of their own
    fix: >-
      Run `lyx loom insert-card` to add a card before the current card 11 holding the shuttleengine mechanism:
      Create `internal/shuttleengine#ModelKey`, `internal/shuttleengine#ModelKey.Wrap` and `internal/shuttleengine#startRequirementsError`;
      Edit `internal/shuttleengine#Runner.checkRequirementsAtStart`, `internal/shuttleengine/doc.go` and `internal/shuttleengine#TestRunner_Requires`, with the four-form way-forward text and its `TestRunner_Requires` rows moved from the Intent and Tests of the seat card.
      Remove those targets and paragraphs from the seat card, which then uses `ModelKey` from the earlier card.
      After the renumbering update the prose card references that shift (card 1's mention of card 13's scan, card 13's `cards 1 to 10` stays valid) and re-run `lyx loom validate-plan`.
      The ModelKey Create targets keep their current form, so no new target check is involved.
---

### [BLOCKING] Card 8 leaves the webster entry of the shed recipe on the removed field

**Location:** `_lyx/plan/08-webster-session-engines.md:12`
**Issue:** Card 8 replaces `RunDeps.Engine` with `Engines` and lists `shedfake#WebsterSeams`, but `internal/shedrecipe/entries_simple.go` reads `env.WebsterDeps.Engine` in `websterEntry` (`requireSeam("Webster", "WebsterDeps.Engine", ...)`), so the package stops compiling at card 8.
Its tests name the field too: `zeroEnvField` has a `WebsterDeps.Engine` case, `simpleEntryCases` lists it in `validatedFields`, and `TestWebsterEntry_SeamFields` iterates it.
`internal/battencli/wire_seams_test.go` keys `intentionallyNil` on `WebsterDeps.Engine`, and `envkit.NilSeams` walks `Env` by reflection, so after the rename the batten wiring test reports `WebsterDeps.Engines` as an unlisted nil.
None of these five is a target of card 8 or any other card.

### [BLOCKING] Card 10's new func field is an unwired seam to the env walkers

**Location:** `_lyx/plan/10-merriam-instruction-files.md:6`
**Issue:** `RunDeps.InstructionFiles` is a func field whose nil means none.
`shedfake.WebsterSeams` (so `envkit.FullEnv`) leaves it nil, and batten's `wire` leaves it nil as well.
`envkit.NilSeams` reports every nil func in `Env`, skipping only the paths in `nilLegal`, and `envkit_test.go` requires `NilSeams(FullEnv(t))` to be empty while `battencli`'s `TestWire...` seam test requires every reported path to be listed.
Card 10 touches neither `nilLegal` nor `intentionallyNil`, so both tests fail at card 10.

### [MEDIUM] `lyx webster run` never receives the instruction files

**Location:** `_lyx/plan/10-merriam-instruction-files.md:29`
**Issue:** `internal/webstercli/run.go` builds `RunDeps` field by field in `websterCLI.runDeps`.
Card 10 says both wiring prologues set the CLI's lister and that `executionBatches` and `rebaselineCmd` call it, and it says `loomCLI.wire` fills `RunDeps.InstructionFiles`; it never says `runDeps` copies it.
With the field left nil, `Run`'s partition formation and `autoRebaseline` count no instruction file, so the `run` verb sizes Master's start base differently from today's `CLAUDE.md` count and from `validate --batches` on the same tree, against D7.
No planned test would notice: the existing `Run` tests leave the field nil by design.

### [LOW] The unwrapped Start way forward does not fit `lyx orch start`

**Location:** `_lyx/plan/11-seat-requires.md:39`
**Issue:** The only caller with `ForkSubagents` that wraps no `ModelKey` is `lyx orch start`: `orchSpec` sets `ForkSubagents: true`, passes `orch.yaml`'s `model` through verbatim and sets no engine, so it runs on `shuttle.yaml`'s default.
The card keeps today's generic clause for it, `give the role a model alias of an engine that meets them`, and claims no requirements refusal leaves any Start caller without a way forward.
The orch has no alias to give, so the step named does not work for its value shape; the working step is `engine:` in `shuttle.yaml`, the form the card already writes for `lyx burler run`.
Round 1's F4 had the refusal lose its way forward entirely; the fixer kept the generic clause, and this finding is only that the kept wording names a step the orch cannot take, not a re-raise of the dropped clause.
Unreachable while `claude` is the only engine, which is why this is LOW.
D8 calls `lyx burler run` the one caller that resolved no key, a premise the orch also breaks; the finding is raised against the plan's claim, not the record.

### [LOW] The `master engine not registered` row over-claims `run` and leaves a doubled way forward open

**Location:** `_lyx/plan/10-merriam-instruction-files.md:39`
**Issue:** I traced each caller of `executionBatches` and `MerriamBase` in `internal/webstercli`.
`begin-batch`, `record-batch`, `recover-batch` and `rebaseline` refuse `no run in progress` before reaching them, `run` goes straight to `websterengine.Run`, and only `validate` and `await-batch` reach `executionBatches` with a nil state.
So `a verb run with no state` in the row's trigger includes `run`, which card 9's entry gate refuses first under the other row, and the tests name `validate --batches` and `rebaseline` only.
Both webstercli callers currently append `; way forward: transient, re-run the verb` to a `MerriamBase` failure; card 10 does not say they return the lister's error without it, and its test rows assert only that the message names `master`.
An implementation that puts the lister call under the same wrapper emits two contradictory way forwards on a correctness-halt row.

### [LOW] An existing refusal row keeps naming files `MerriamBase` no longer names

**Location:** `_lyx/plan/10-merriam-instruction-files.md:39`
**Issue:** `contracts/specs/refusal-spec.md` has the row `Merriam start base unreadable`, whose trigger is a verb that cannot read a present `CLAUDE.md` or `CLAUDE.local.md`.
After card 10 the files are whatever Master's engine names, so the row goes stale; the card edits the same file but only adds a row.

### [LOW] Card 8 states no bound for the engine-less read

**Location:** `_lyx/plan/08-webster-session-engines.md:77`
**Issue:** The engine-less branch of `sessionAuditEngine` reads a pre-change session through the current default.
Card 8 states only the refusal when that default has no fork audit.
D6's Bound says that when the default differs from the engine that ran the session the read either ends in that engine's own audit error or returns an empty audit, which can pass a run-exit audit or hide a run's own path from `reset`, accepted while one engine is registered.
The rubric's attack-surface question wants that let-through and its bound in the introducing card's Intent.

### [LOW] `ModelKey` and its `Wrap` deserve their own card

**Location:** `_lyx/plan/11-seat-requires.md:3`
**Issue:** The rubric gives a symbol that is part of a module's public surface, or reused across packages, its own card.
`ModelKey` is public in `shuttleengine` and used by `seatengine`, `burlerengine`, `loomengine`, `shedrecipe` and `burlercli` in card 11 and by `websterengine` and `loomcli` in card 12.
Card 11 bundles the shuttle mechanism with the seat requirements, the loom keys, burler's reviewer key and five refusal rows.

## Rejected

- Focus 1, fixture order between card 1 and the fixture-moving card: not a defect.
  Card 1 itself drops `Engine` `claude` from the role literals of `newRunFixtureOver`, `newRecoverFixtureOver` and `newVerbsFixture`, the only three fixtures that feed `Run`'s Master spec or `recoverSpawn`'s recovery spec (`RunDeps{` and `RecoverDeps{` have no other test constructor, and `shedfake.WebsterSeams` is a placeholder never called), so the commit that makes the specs carry the resolved engine also fixes every fixture they reach.
  Card 7 then moves the same three to `ModelChoice` values keeping the empty engine.
- Focus 2, the row against card 9's gate: the two planned webstercli rows (`validate --batches`, `rebaseline` after a seeded state) never enter `websterengine.Run`, so the gate does not pre-empt them; the wording and way-forward defects are F5.
  `Run`'s own lister error is dead after the gate and card 10 already says so.
- Focus 3, the key shapes: `discussion`, `plan`, `driver` and `master` are scalar model-specs, `fan_review` takes a scalar or a per-round list (`internal/configcli/doc.go`), and `discussion_advisors` is a `{name, model}` list that card 11 handles with the entry form.
  `lyx config <module> --set` with no `--layer` writes the worktree file, which the default-layer guard does not refuse from a task session.
  Each Requires row has a planned test through a real `Runner` over a fake engine set (card 11's seat tables, advisor, `lyx burler run`, card 12's Master and driver arm).
  The only caller left with a wrong step is the orch (F4).
- Fidelity, order and verify coverage: every Decision, Constraint and acceptance criterion of the decision record maps to a card, each card's dependencies come from earlier cards, and the verify section runs every package's untagged and integration tests.
