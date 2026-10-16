# Card 3 — twill-fixer-engine

**Create:**
- `contracts/stencils/burler/burler-fix-self-check.md`
- `contracts/stencils/burler/burler-fix-check-rubric.md`
- `plan:contracts/stencils#BurlerFixSelfCheck` -> `var BurlerFixSelfCheck []byte`
- `plan:contracts/stencils#BurlerFixCheckRubric` -> `var BurlerFixCheckRubric []byte`
- `plan:internal/burlerengine#selfCheckBlock` -> `func selfCheckBlock(stencilsDir string, p *Profile) (string, error)`
- `plan:internal/burlerengine#forkViolation` -> `func forkViolation(audit *shuttleengine.ForkAudit) error`
- `plan:internal/burlerengine#auditFixRound` -> `func auditFixRound(audit *shuttleengine.ForkAudit, wantN int) (warnings []string, forks int, err error)`
- `plan:internal/burlerengine#twillEngineRefusal` -> `func twillEngineRefusal(opts RunOpts, err error) error`

**Edit:**
- `contracts/stencils/burler/burler-step-fix.md`
- `contracts/stencils#entries`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `contracts/specs/refusal-spec.md`
- `internal/burlerengine#Profile`
- `internal/burlerengine#RunOpts`
- `internal/burlerengine#FixResult`
- `internal/burlerengine#Profile.validateReview`
- `internal/burlerengine#Profile.validateFix` -> `func (p *Profile) validateFix(worktreeRoot string, cfg Config) error`
- `internal/burlerengine#Profile.fixTable`
- `internal/burlerengine#composeFixPrompt`
- `internal/burlerengine#auditClusterRound`
- `internal/burlerengine#Engine.Fix`
- `internal/burlerengine#Engine.ProbeFix`
- `internal/burlerengine#Engine.ResumeFix`
- `internal/burlerengine#Engine.concludeFix`
- `internal/burlerengine#TestAuditClusterRound`
- `internal/burlerengine#TestProfile_Validate`
- `internal/burlerengine#TestEngine_Fix_TableShapeAndFiles`
- `internal/burlerengine#TestEngine_Fix_Outcomes`
- `internal/burlerengine#TestTemplate_FillsWithAllMarkers`
- `internal/burlerengine#TestTemplate_OptionalDirectives`
- `internal/burlerengine#fixStepMarkerValues`
- `internal/burlerengine/doc.go`

**Uses:**
- `plan:internal/burlerengine#lensList`
- `internal/burlerengine#newFixCase`
- `internal/burlerengine#testClusterFanConfig`

**Intent:**
Build the twill fixer in the engine (D7, D9): a fix half whose profile names a fixer fan spawns its fixer with fork subagents authorized and has it check its own change set with read-only forks before its fixer-report.
A profile with no fixer fan is today's single fixer, byte for byte.

`Profile` gains `FixFan` (a fan name; empty is the single fixer) and an unexported `fixLenses`.
`validateFix` takes the engine's `Config` and resolves a non-empty `FixFan` through `ResolveFan` into `fixLenses`, so an unknown fixer fan fails the half before any spawn; its three callers, `Engine.Fix`, `Engine.ProbeFix` and `Engine.ResumeFix`, pass `e.cfg`.
A `ResolveFan` error in `validateFix`, and in `validateReview` for `ClusterFan`, is wrapped with `; way forward: define the fan in burler.yaml, then resume the run or re-run the verb`, since card 5 hands each half a fan recorded at spawn time, which an edit of `burler.yaml` before the fix half or a resume can leave undefined.
`RunOpts` gains `FixKey`, a `shuttleengine.ModelKey` naming the key the fixer's model came from as `ReviewKey` does for the reviewer, and `FixFanKey`, the name of the key or profile field the fixer fan came from.
`fixTable` sets the fixer seat's `ForkSubagents` exactly when `FixFan` is non-empty and its `ModelKey` to `opts.FixKey`, and its doc comment drops "never forks" for "forks exactly when the profile names a fixer fan".
A forking fixer still carries its row's gates: a turn end whose Stop payload reports an outstanding fork is a wait, so no gate runs while a fork is live, as for the gated Plan-Write lead that forks under `plan_lenses`; card 7 re-states that premise in `internal/loomrecipe`'s gate-quiescence test.

Two new stencils, embedded and registered after card 2's:
`burler-fix-self-check` is the bench's self-check block (`docs/research/burler-twill-fixer-bench-2026-10-16/setup/selfcheck-block.md`) with markers `{{.rubric_path}}` and `{{.lens_list}}`, led by its heading and ending in a blank line;
its step 4 judges each fork finding by pointing at the fix step's Fix-everything rule instead of restating the three dispositions, its step 5 records the self-check in a `## Self-check` section of the fixer-report, one line per fork finding, naming a fork that returned no report, and it keeps the bench's one-fork-pass rule, adding that a fixer re-prompted by its gate or resumed after its forks were lost spawns no further forks and names each lost fork as "no report".
`burler-fix-check-rubric` is the bench's `setup/fix-check-rubric.md`, deployed so the fixer reads it at `stencilstore.Path(stencilsDir, "burler-fix-check-rubric")`.
`burler-step-fix.md` gains the optional marker `{{.self_check}}` directly before `## Fixer-report rule`, with no line of its own, so an empty value adds no byte.
`selfCheckBlock` returns "" for an empty `FixFan` and otherwise the part filled with the rubric path and `lensList(p.fixLenses)`; `composeFixPrompt` fills `self_check` as optional beside `friction_directive`.
The self-check runs in every fixer session of a twill round, a gate-findings-only fix included, because both reach `composeFixPrompt`; a skipped fix spawns no session and so runs none.
`prompt.go`'s file comment names the self-check part among the fix half's reads.

`auditClusterRound` keeps its order and messages, its per-fork checks moved into `forkViolation`, which returns the first Agent-call, write, mutating-git or named-spawn violation in today's order.
`auditFixRound` calls `forkViolation` first and wraps a violation as `burler: twill fixer <violation>; way forward: inspect the target's working-tree changes, revert what the fork wrote, and resume the run`;
it then counts the forks, and a count unequal to `wantN`, none included, is a warning `twill fixer spawned N fork(s) for M lens(es)`, logged and returned, never an error, beside the never-returned-report warnings.
`FixResult` gains `Lenses` (the fixer fan's lens names in fan order, nil for a single fixer), `Forks` (the audited count) and `Warnings`.
`concludeFix`, for a done fixer of a twill round, runs `auditFixRound(fixer.ForkAudit, len(p.fixLenses))` before the gate check, so a violation fails the half whether or not the gate passed and a fork that wrote over a passing gate still stops the run; the audit reads the session's whole fork record, so it holds across the gate's re-prompts of the same session.
A start error wrapping `shuttleengine.ErrRequirementsUnmet` for a twill round goes through `twillEngineRefusal` before the generic `burler: shuttle run:` wrap.
It reads one wording with the two keys as slots, `burler: the twill fixer's fan from <fan key> needs fork subagents, which the engine of <model key> cannot run: <refusal>; way forward: <step>`, whose step follows D7 per caller.
For a set `FixKey.Key` (a shed row), the model key is `<key> in <module>.yaml` and the step is `move <key> in <module>.yaml to an engine with forks, then resume; setting the <fan key> entry to "" reaches only rounds whose reviewer is spawned after the edit`.
For a `DefaultEngine` key (`lyx burler run`), the model key is `engine: in shuttle.yaml` and the step is `set engine: in shuttle.yaml to an engine with forks, or remove <fan key> from the profile, and re-run`.
Only a fix row told a fixer fan key reads a recorded fixer fan (card 5), and `lyx burler run` sets its keys (card 4), so a twill fixer always carries one of these two key shapes.
The input that reaches it is a twill profile run through a shuttle whose start refuses for missing forks; a single fixer's start refusal keeps today's message.

Tests:
`TestProfile_Validate` passes a `Config` to `validateFix` and gains fix-half rows for an unknown `FixFan` (refused naming the fan and the way forward) and a resolving one (`fixLenses` set in fan order), using `testClusterFanConfig`; its existing unknown-`ClusterFan` row asserts the way forward too.
`TestAuditClusterRound` gains `auditFixRound` rows in its table shape: exact count clean, none, fewer and more forks than lenses each a count warning with no error and the counted forks, a never-returned report a warning, and a write-call fork the twill-fixer violation with its way forward; its existing rows pin `auditClusterRound` unchanged through `forkViolation`.
`TestEngine_Fix_TableShapeAndFiles` becomes a two-row table: the single row keeps today's assertions plus that the fix step equals the fix stencil filled with the `{{.self_check}}` literal removed, and that `Lenses` is nil;
the twill row sets `FixFan` and `opts.FixKey`, and asserts `spec.ForkSubagents`, the seat's model key, that the fix step differs from the single render and carries the rubric path and the filled lens bullets in fan order, and `FixResult.Lenses`; it asserts no other stencil text.
`TestEngine_Fix_Outcomes` gains rows: a twill fixer whose done result carries a write-call fork audit returns the violation error with the way forward and its gate, which proves `concludeFix` wires the audit; one with a nil audit returns no error with `Forks` 0 and a count warning;
a twill start error wrapping `ErrRequirementsUnmet` under a `loom` `FixKey` names that key, the `FixFanKey` and the resume step, under a `DefaultEngine` key names `engine: in shuttle.yaml`, `remove fix-fan from the profile` and `re-run`, and a single fixer's start error wrapping `ErrRequirementsUnmet` keeps today's `burler: shuttle run:` message with no fan key.
`fixStepMarkerValues` gains `self_check`, `TestTemplate_FillsWithAllMarkers` lists it as optional on the fix step, and `TestTemplate_OptionalDirectives`' fix-step subtest adds `self_check` to its `FillOptional` optional list and renders an empty `self_check` cleanly too.
`TestStencils_TopLevelMarkers` gains `burler-fix-self-check` with `lens_list` and `rubric_path`, and `burler-fix-check-rubric` with none.

`refusal-spec.md` gains three rows in the loom section, beside the review rows: the twill fixer's engine refusal, quoting the one wording with its two slots and the two steps, the twill fixer's fork violation with its way forward, and a recorded fan that no longer resolves with its way forward;
the Out of scope paragraph names them as the exceptions to "Burler's own round errors stay out".
`internal/burlerengine/doc.go` documents the fixer fan, its self-check, the fixer audit's error and warning split and the new result fields.

**ImpactSummary:** `validateFix` has three callers in `fix.go` and one test; `fixTable`, `composeFixPrompt` and `concludeFix` change only for a profile with a fixer fan, which no caller sets until cards 4 and 5, so every current round keeps its behaviour.
