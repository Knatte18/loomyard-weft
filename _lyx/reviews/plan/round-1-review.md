---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:affected-set
    location: _lyx/plan/10-round-gate-tiers.md (Intent, the shedrecipe.Env paragraph)
    summary: Card 10 makes the verify gate entry require the new round seam, but only card 14 fills it in loomcli's production wiring, so from card 10 to card 13 every loom and darn verify row fails requireSeam when the recipe builds.
    fix: Add `internal/loomcli#loomCLI.wire` and `internal/loomcli#loomCLI.wireDarn` to card 10's Edit list, and have card 10's Intent fill `Env`'s round seam there, built from `gateslot.LoadConfig` over the pair's config dirs and `TierList("round")`. Change the sentence "envkit.FullEnv fills it with a fixed round list, and the wiring fills it in card 14" to say card 10 fills both. Drop the seam-filling from card 14's Intent, keeping card 14's base-command and landing renders. This fix passes `lyx loom validate-plan --overlay`.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:way-forward
    location: _lyx/plan/02-gate-tier-config.md (Intent, the LoadConfig outside-every-hub sentence); _lyx/plan/00-overview.md (Shared Decisions, the outside-every-hub bullet)
    summary: Outside every hub, the refusal for a worktree with no gate.yaml names lyx config gate as the step that writes the worktree's own file, but configcli selects LayerDefault for a hub-wide module with an empty --layer, which does not apply without a board, and refuses --layer worktree, so the step cannot succeed (PATTERN-refusal-way-forward), and cards 8 and 13 inherit the dead end.
    fix: Add `internal/configcli#configDirs.selectedLayer` to card 2's Edit list. Have card 2's Intent make an empty `--layer` on a hub-wide module select `LayerWorktree` when the dirs tell no board, so `lyx config gate` writes the standalone worktree's own `gate.yaml`. Add a `TestLoadConfig` row, or a configcli row in card 2's tests, where `lyx config gate --set` in a standalone worktree writes that file and a following `LoadConfig` reads it. The overview's Shared Decision bullet then stands as written. This fix passes `lyx loom validate-plan --overlay`.
  - id: F3
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/plan/12-verifytree-unslotted.md (Intent, first paragraph)
    summary: Card 12 drops verifytree's slot and 60-minute timeout on the premise that every command Verify runs is a lyx gate test --wait call, but webster's plan-level gate and lyx webster verify move to the final gate only in card 13, and Merge-In, Finalize and Publish only in card 14, so from card 12 on those sites run the plan's raw verify section unslotted and unbounded.
    fix: Move card 12 to after card 14 with `lyx loom remove-card` and `lyx loom insert-card`. Cards 13 and 14 then call `verifytree.Verify` with today's pool and timeout arguments, and the moved card drops them at every site it already lists. Or, if the order stays, keep the slot and timeout in card 12 for any command that is not one `lyx gate test --wait` call or such calls joined by `&&`, and retire them in card 14.
  - id: F4
    severity: MEDIUM
    class: scope
    origin: lens:guard-reach
    location: _lyx/plan/09-gate-scopes.md (Intent, the guard-step sentence and the --base bullet; Tests, the --base row)
    summary: Card 9 runs the guard step only for a computed or card scope, and defines computed as no scope flag, so a --base call runs no guard step, but the round and darn writer gates run --base (card 10), and D5's rationale is that the round gate keeps the guard coverage it has today.
    fix: In card 9's Intent, state that under the `go` scoper a `--base` scope also runs the guard step, with the guard packages `DeriveTargets` names, or a `ModuleGuardTests` scan when it falls back to whole. Extend the guard-step sentence to read "a computed, `--base` or card scope". Have the `TestGateTest_Scopes` `--base` row assert the guard step runs through `guard_tier`'s slots.
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/09-gate-scopes.md (Tests, the two-module row of TestGateTest_Scopes)
    summary: The row a two-module change running the tier once per module, with a timeout_min tier killed across the two invocations, cannot happen under a computed scope, since card 5's DeriveTargets falls back to whole for any nested-module file, so the acceptance criterion on timeouts across per-module invocations is never reached.
    fix: Reword the row to a `--card` run over a card whose targets span the root module and a nested module (card 6's `CardPackages` groups them per module). The tier runs once per module, and a `timeout_min` tier is killed across the two invocations with the timeout line. Add a computed-scope row where a nested-module file falls back to whole, naming the reason on stderr.
  - id: F6
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lyx/plan/19-non-tier-holders.md (Edit list and Forced edits)
    summary: Card 19 deletes Limits.GoParallel but does not target internal/gateslot/testmain_test.go, whose singleSlotLimits builds Limits{Slots 1, GoParallel 4}, so gateslot's test package stops compiling.
    fix: Add `internal/gateslot#singleSlotLimits` to card 19's Edit list. Add to its Forced edits that `singleSlotLimits` drops `GoParallel`, setting `Tier0Parallel` only where a test reads it. This fix passes `lyx loom validate-plan --overlay`.
  - id: F7
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lyx/plan/14-landing-gates.md (Intent, the darnengine.LoadConfig refusal; Tests)
    summary: Card 14's darn loader refuses a tree_scope_verify lacking --wait, but configcli's TestDispatchHubWideBoard seeds and saves lyx gate test --tags integration --guards {targets} through the darn editor rows (internal/configcli/configcli_test.go:680), and no card targets it.
    fix: Add `internal/configcli#TestDispatchHubWideBoard` to card 14's Edit list. Add to its Tests that the test's `darnTreeVerify` constant becomes `lyx gate test --wait --tier 1 {targets}`. This fix passes `lyx loom validate-plan --overlay`.
  - id: F8
    severity: MEDIUM
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/04-gate-run-log.md (Intent, TierTimeoutLine); _lyx/plan/12-verifytree-unslotted.md (Intent, landingshed's Stuck reason)
    summary: The timeout way forward says to raise the tier's timeout_min with lyx config gate, whose natural form --set tiers.tier1.timeout_min=90 writes an entry literally named tier1.timeout_min (yamlengine splitOpenMapKey keeps the rest of the key as the entry name), which card 2's loader refuses as a tier key not spelled tier<N>.
    fix: In card 4's `TierTimeoutLine` sentence and card 12's landingshed Stuck-reason sentence, name D1's whole-tier form. The step becomes rewriting the tier whole with `lyx config gate --set 'tiers.tier<N>={...}'` carrying every key of the tier and the raised `timeout_min`, from the prime. Assert that wording in `TestWatchdogLine`'s `TierTimeoutLine` row and `TestVerifyGate_CheckMapsVerifyResult`.
  - id: F9
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lyx/plan/11-card-verify-runner.md (Intent, the lyx gate card-verify paragraph)
    summary: Card 11 gives lyx gate card-verify the operator audience annotation, but the verb is run by the Webster implementer fork as its prompt names it (card 15), which internal/clihelp's AudienceRole marks, so the command index lists it under the wrong audience.
    fix: In card 11's Intent, replace "with the operator audience annotation" with "with the role audience annotation (`clihelp.AudienceRole`), since the implementer fork's prompt names it". Have `TestGateCardVerify`, or the `cmd/lyx` CLI-tree walk it already passes, see the role annotation.
  - id: F10
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/05-impactset-tree-targets.md (Create, DeriveTargets); _lyx/plan/09-gate-scopes.md (Intent, computeScope)
    summary: The pre-tier ceiling is meant to bound a stalled go list, but DeriveTargets takes no context and today's listPackages starts go list with a plain exec.Command, so the ceiling can only abandon the work and leaves the stalled child running after the call fails.
    fix: Re-sign card 5's Create line to `func DeriveTargets(ctx context.Context, worktree, base string) (Targets, error)`. State in card 5's Intent that `go list` starts with `exec.CommandContext` under that context and its git reads stop when it is cancelled. Card 9's `computeScope` passes the pre-tier ceiling's context. This fix passes `lyx loom validate-plan --overlay`.
  - id: F11
    severity: LOW
    class: scope
    origin: lens:affected-set
    location: _lyx/plan/12-verifytree-unslotted.md (Edit list and Intent, the callers paragraph)
    summary: Once loomshed's NewVerifyGate loses slots, shedrecipe.Env.GateSlots has no reader left, but card 12 removes only Landing.GateSlots, leaving the Env field, loomcli's wire line that sets it, TestWire_DefaultConfig's assertion and envkit's nilLegal GateSlots entry.
    fix: Add `internal/shedrecipe#Env`, `internal/loomcli#loomCLI.wire` and `internal/loomcli#TestWire_DefaultConfig` to card 12's Edit list. Have its Intent delete `Env.GateSlots`, the line in `loomCLI.wire` that sets `GateSlots` from `hubgeom.GateSlots(location)`, `TestWire_DefaultConfig`'s assertion of it, and envkit `nilLegal`'s `GateSlots` entry beside `Landing.GateSlots`. This fix passes `lyx loom validate-plan --overlay`.
  - id: F12
    severity: LOW
    class: scope
    origin: lens:affected-set
    location: _lyx/plan/19-non-tier-holders.md (Edit list)
    summary: docs/benchmarks/running-tests.md, a how-to guide rather than a measurement report, keeps lyx gate test --tags <tags> <packages> (lines 89-90), which card 8 refuses, and numbers the tiers 1 to 4 against card 19's tier0 to tier3 in PATTERN-test-speed.md.
    fix: Add `docs/benchmarks/running-tests.md` to card 19's Edit list. State in its Intent that the guide's tier section moves to the hub's tier numbers, `tier0` untagged through `tier3` `llm`, and its gate lines name `lyx gate test --tier <n>` in place of `--tags`. This fix passes `lyx loom validate-plan --overlay`.
  - id: F13
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/plan/03-gate-slot-drain.md (Intent, the ascending-acquisition sentence)
    summary: Card 3 bounds the drain wait by a single-slot holder always finishing within its own tier timeout, but the non-tier holders hold a slot with no tier timeout, cmd/testtiming for its whole run in every mode (card 19), so a drain waiter, and every single-slot acquire queued behind it, can wait for a whole timing run.
    fix: Reword card 3's bound sentence. A tier holder finishes within its own tier timeout, and a card `**Verify:**` holder within the runner's timeout. The plan glyph type load is bounded by its load, and `cmd/testtiming`, which the operator runs by hand, is bounded only by its own run, so a drain waiter queued behind a timing run waits for it.
  - id: F14
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/plan/08-gate-tier-verb.md (Tests, TestGateTest_Scenario's covered-tier row); _lyx/plan/09-gate-scopes.md (Tests, TestPlanTierRuns' tier1 skip row)
    summary: TestGateTest_Scenario's row --tier 0 --tier 1 with tier1 covering [0] running only tier1 repeats TestPlanTierRuns' covered-tier row, and card 9's TestPlanTierRuns row with the guard step skipped when the call lists tier1 repeats card 8's row with the guard step skipped when the call runs guard_tier whole (PATTERN-test-economy).
    fix: In card 8's Tests, drop the `--tier 0 --tier 1` covered row from `TestGateTest_Scenario` and leave it to `TestPlanTierRuns`' covered-tier row. In card 9's Tests, drop the "that guard step skipped when the call lists `tier1`" row, which card 8's `TestPlanTierRuns` row already covers, and keep the whole-fallback row that plans the guard step.
  - id: F15
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/plan/15-webster-prompt-gates.md (Tests, TestCardGateCommand)
    summary: TestCardGateCommand keeps its nested-module and delete-only rows, but card 15 renders the same single --card call for every card, so those rows pin no behavior the plain-card row does not, while TestCardPackages (card 6) and TestGateTest_Scopes (card 9) own the package rule.
    fix: In card 15's Tests, cut `TestCardGateCommand` to the one row of a card's gate call followed by its two lints, plus the dangling `card` list refusal. Drop the nested-module and delete-only rows, whose packages `TestCardPackages` (card 6) covers.
  - id: F16
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/plan/16-retire-verify-tiers.md (Intent, the VerifyTiersValue sentence)
    summary: Card 16 keeps languageconfig.VerifyTiersValue to render rename_tools alone, so the function keeps the name of the key this card retires, and its name no longer says what it renders.
    fix: Insert a Rename card after card 16 with `lyx loom insert-card`, renaming `internal/languageconfig#VerifyTiersValue` to `internal/languageconfig#BulletListValue`. In card 16's Intent, change "`VerifyTiersValue` keeps rendering `rename_tools`, its doc saying so" to name the renaming card.
---

### [BLOCKING] Card 10's round seam has no production filler until card 14

**Location:** `_lyx/plan/10-round-gate-tiers.md`, Intent, the `shedrecipe.Env` paragraph.

**Issue:** Card 10 says `shedrecipe.Env` gains the `round` seam, "which the verify gate entry requires and passes to `NewVerifyGate`".
It also says "`envkit.FullEnv` fills it with a fixed round list, and the wiring fills it in card 14".
The verify entry checks its seams with `requireSeam` (`internal/shedrecipe/entries_gate.go:236`).
The production Env is built at `internal/loomcli/wiring.go:696-720` for loom and at `internal/loomcli/darnwiring.go:269` for darn, and card 10 targets neither.
Every tree from card 10 through card 13 therefore fails to build any recipe that holds a verify row.
That breaks `lyx loom start` and `lyx loom run` there, and the loomcli tests that build producers.
Card 12 targets `internal/loomcli`, so its own gates run those tests on that tree.
A card that depends on something a later card does breaks the plan spec's "Plan vs. schedule" rule.

### [BLOCKING] The standalone no-`gate.yaml` refusal names a step that cannot write the file

**Location:** `_lyx/plan/02-gate-tier-config.md`, Intent; `_lyx/plan/00-overview.md`, Shared Decisions.

**Issue:** Card 2 says `LoadConfig` outside every hub, with no worktree `gate.yaml`, names "`lyx config gate`, which writes the worktree's own file there".
`gate` is hub-wide, so `selectedLayer` maps an empty `--layer` to `LayerDefault` (`internal/configcli/hubwide.go:102-103`).
`LayerFile(LayerDefault)` applies only when `dirs.Board` is set (`internal/configengine/layers.go:107-109`), so the verb answers that the layer "does not apply here".
`--layer worktree` on a hub-wide module is refused outright (`hubwide.go:106-107`).
No card targets `internal/configcli`, so the step fails from the state the refusal leaves.
Card 8 inherits the dead end for a standalone gate call, and so does card 13 for `standalonegeom.WebsterGeometry`, a webster refusal `PATTERN-refusal-way-forward` binds.

### [MEDIUM] Card 12 retires verifytree's slot and timeout before every site renders a gate call

**Location:** `_lyx/plan/12-verifytree-unslotted.md`, Intent.

**Issue:** Card 12 states that "every command it runs from here on is one `lyx gate test --wait` call".
Webster's plan-level gate and `lyx webster verify` still run the plan's `## verify:` section until card 13.
Merge-In, Finalize and Publish still run it until card 14, since the loom verify source reads the plan there (`internal/loomcli/wiring.go:705`, `planVerifySource`).
Between card 12 and card 14 those raw commands run through `verifytree.Verify` with no slot and no timeout.
Card 12's stated bound therefore depends on two later cards.

### [MEDIUM] `--base` runs no guard step, so the round gate loses its guard coverage

**Location:** `_lyx/plan/09-gate-scopes.md`, Intent and Tests.

**Issue:** Card 9 defines "Computed (no scope flag, no targets)" and then runs the guard step only for "a computed or card scope".
Its `--base` row asserts no guard step.
The round and darn writer gates run `--base` (card 10).
Card 10 deletes the round gate's Go-side guard scan and relies on the gate's own scope.
D5's rationale says the round gate runs the guard tests today and the scoper keeps that coverage, so as planned the round gate drops them.

### [MEDIUM] The two-module computed-scope row cannot happen

**Location:** `_lyx/plan/09-gate-scopes.md`, Tests.

**Issue:** Card 5's `DeriveTargets` falls back to whole for any file of a nested module, since the root `go list ./...` does not list it (`internal/impactset/derive.go:127`).
So a computed scope never yields two modules.
The row's per-module invocations and the timeout killed across them cannot be reached as written.
That leaves the acceptance criterion "A tier exceeding `timeout_min` across its per-module invocations is killed" with no reaching test.

### [MEDIUM] Card 19 misses gateslot's test-main `Limits` literal

**Location:** `_lyx/plan/19-non-tier-holders.md`, Edit list.

**Issue:** `internal/gateslot/testmain_test.go:43` returns `Limits{Slots: 1, GoParallel: 4}` from `singleSlotLimits`.
Card 19 deletes `Limits.GoParallel` and lists other forced test edits, but not this one.
So the package's tests stop compiling at card 19.

### [MEDIUM] The darn `--wait` refusal breaks configcli's darn editor fixture

**Location:** `_lyx/plan/14-landing-gates.md`, Intent and Tests.

**Issue:** `TestDispatchHubWideBoard` seeds and saves `tree_scope_verify: "lyx gate test --tags integration --guards {targets}"` through the darn editor rows (`internal/configcli/configcli_test.go:680`).
Card 14's loader refuses that value because it lacks `--wait`.
No card targets the test.

### [MEDIUM] The timeout way forward leads to a key the loader refuses

**Location:** `_lyx/plan/04-gate-run-log.md`, Intent; `_lyx/plan/12-verifytree-unslotted.md`, Intent.

**Issue:** "Raise its `timeout_min` with `lyx config gate`" invites `--set tiers.tier1.timeout_min=90`.
`splitOpenMapKey` keeps everything after the open-map prefix as the entry name (`internal/yamlengine/set.go:210-216`).
That writes an entry named `tier1.timeout_min`, which card 2's loader refuses as a tier key not spelled `tier<N>`.
D1 writes a tier whole with one flow-mapping `--set`, and the way forward should name that form.

### [MEDIUM] `lyx gate card-verify` carries the wrong audience

**Location:** `_lyx/plan/11-card-verify-runner.md`, Intent.

**Issue:** `pattern/PATTERN-cli-cobra.md` requires every invocable command to carry an audience annotation from `internal/clihelp`'s closed set.
In that set, `AudienceRole` marks "a command a spawned role agent runs, named by that role's prompt" (`internal/clihelp/annotations.go:27-28`).
The implementer fork's prompt names `lyx gate card-verify` (card 15), and the card says so, yet it annotates the verb operator.
The command index lists a command only under the audience it names, so the role index would miss it.

### [LOW] The pre-tier ceiling cannot stop the work it bounds

**Location:** `_lyx/plan/05-impactset-tree-targets.md`, Create; `_lyx/plan/09-gate-scopes.md`, Intent.

**Issue:** `DeriveTargets(worktree, base string)` takes no context.
Today's `listPackages` uses a plain `exec.Command("go", "list", ...)` (`internal/impactset/derive.go:127`).
When the ceiling fires, the call fails, but a stalled `go list` keeps running.
Card 8 nonetheless says a re-run passes a transient hang.

### [LOW] `shedrecipe.Env.GateSlots` is left with no reader

**Location:** `_lyx/plan/12-verifytree-unslotted.md`, Edit list.

**Issue:** `internal/shedrecipe/entries_gate.go:239` is the field's only reader, and card 12 removes that argument.
The field stays, still set at `internal/loomcli/wiring.go:706`, asserted in `TestWire_DefaultConfig` and listed in envkit's `nilLegal` (`internal/testkit/envkit/envkit.go:45`).

### [LOW] The running-tests guide keeps the retired flag and the old tier numbering

**Location:** `_lyx/plan/19-non-tier-holders.md`, Edit list.

**Issue:** `docs/benchmarks/running-tests.md:89-90` names `lyx gate test --tags <tags> <packages>`, which card 8 refuses.
Its tier section numbers the tiers Tier 1 to Tier 4.
Card 19 moves `PATTERN-test-speed.md` to `tier0` through `tier3`, so the two disagree.

### [LOW] The drain bound overstates what bounds a held slot

**Location:** `_lyx/plan/03-gate-slot-drain.md`, Intent.

**Issue:** "A single-slot holder always finishes within its own tier timeout" holds for tier runs only.
Card 19 has `cmd/testtiming` hold one slot for its whole run in every mode, with no tier timeout.
So the bound the card states for the drain's wait is false for that holder.

### [LOW] Two planned rows repeat a covered behavior

**Location:** `_lyx/plan/08-gate-tier-verb.md`, Tests; `_lyx/plan/09-gate-scopes.md`, Tests.

**Issue:** The covered-tier rule is pure run-list logic that `TestPlanTierRuns` already covers.
The scenario test's `--tier 0 --tier 1` row repeats it.
Card 9's "guard step skipped when the call lists `tier1`" row repeats card 8's "skipped when the call runs `guard_tier` whole" row.

### [LOW] TestCardGateCommand keeps rows that no longer differ

**Location:** `_lyx/plan/15-webster-prompt-gates.md`, Tests.

**Issue:** After card 15 the webster side renders one `--card` call per card, whatever the card's directories.
So the nested-module and delete-only rows assert the same output as the plain row.
The package rule they pinned moves to `TestCardPackages` (card 6).

### [LOW] VerifyTiersValue keeps a retired key's name

**Location:** `_lyx/plan/16-retire-verify-tiers.md`, Intent.

**Issue:** After card 16 the function's only caller fills `rename_tools` (`internal/websterengine/render.go:420,524`).
Its name still says it renders the `verify_tiers` key, which this card retires.

## Rejected

- guard-reach: card 8's pre-tier ceiling row names no reaching input. Rejected: card 8 has `runGateHelper` pass the short ceiling, and that injected ceiling is the named input.
- guard-reach: `planTierRuns` refuses a dangling `guard_tier` under the `none` scoper. Rejected: D1 prescribes that use-time refusal, and the template's `guard_tier: 0` is declared whenever a call gets past the no-`tier0` refusal, so only an operator-set dangling value reaches it.
- guard-reach: card 16's standalone `none` rendering never reaches a Webster fork prompt. Rejected: the same wiring branch is reachable through `PlanTable`, and begin-batch refusing a worktree with no file first is D2's rule, so nothing is wrong.
- Card 1's flow-list `--set` shape as unlicensed scope (handler check). Rejected: card 2 makes `gates` an open map, and D12 has the orch set its lists, which a string-scalar `--set` cannot write.
- Card 19 shortening the `PATTERN-recipe-setting-lookup` and `PATTERN-push-both-sides` overview lines (handler check). Rejected: `PATTERN.md` is at 15743 bytes, above its 15564-byte headroom line (`internal/pattern/check.go:24`), and the declared entries' text exceeds the 320-character line cap. The cut is forced, and each rule stays whole in its background file.
