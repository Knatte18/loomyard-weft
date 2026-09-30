---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/plan/06-rotator-prior-plan-amendment.md
    summary: Card 6 makes the rotator render a stencil that loomrecipe's buildSequenceFixture never seeds, so every loomrecipe sequence test that reaches Plan-Write fails, and the card neither targets that fixture nor mentions it in its ImpactSummary
  - id: F2
    severity: LOW
    location: _lyx/plan/01-claudeengine-deny-notice.md
    summary: Card 1's ImpactSummary calls the change additive, but its Intent requires reshaping buildSettings' body around a shared install-condition helper, which neither the summary nor the tests name
  - id: F3
    severity: NIT
    location: _lyx/plan/05-loomengine-prior-plan-block.md
    summary: Card 5 lists internal/loomengine#composePlanPrompt under Uses although PriorPlanBlock never calls or depends on it, which puts a false edge into the derived dependency graph
---

### [BLOCKING] Card 6 breaks the loomrecipe sequence tests and does not target the fixture that needs the new stencil

**Location:** `_lyx/plan/06-rotator-prior-plan-amendment.md` (Edit list and ImpactSummary)

**Issue:**
After card 6, the rotator closure calls `loomengine.PriorPlanBlock(stencilsDir, ...)` whenever the rotation moved at least one file.
Card 5's renderer reads the stencil with `stencilstore.Read`, which never falls back to the embedded default (`internal/stencilstore/reconcile.go:24-34`), and card 6 returns a render failure as the closure's error, so the producer aborts.
`internal/loomrecipe/fixture_test.go`'s `buildSequenceFixture` calls `seedPlanFixture(t, dir, false)`, which writes `00-overview.md` and `01-first-card.md` into the plan directory, and then builds its stencils directory with `seedBouncerStencils`.
That helper (`fixture_test.go:83-102`) seeds only the two bouncer templates and the three loom rubrics, so `loom-template-prior-plan` is absent.
The first Plan-Write `Call` in that fixture therefore rotates the seeded plan, fails to render the block, and returns an error instead of running the fake shuttle.
`buildSequenceFixture` backs 14 tests, including `TestSequence_FullRunBlocksAtPublish`, which today passes through Plan-Write (confirmed by running it on this tree), and the `resume_test.go` and `frictionreflect_test.go` sequence runs.
None of the six cards targets `internal/loomrecipe/fixture_test.go`, so an implementer bound to card 6's target list has no licensed place to fix the failure, and the plan's own `go test` verify step, which the decision record's last acceptance criterion requires to pass, would fail.
Card 6's ImpactSummary checks only construction ("every loom wiring already sets `Env.StencilsDir`"), which holds, and misses this runtime dependency on the stencil being present.

**Fix:**
Add `internal/loomrecipe/fixture_test.go` to card 6's Edit list.
Have card 6 seed `stencils.LoomTemplatePriorPlan` under `loom-template-prior-plan` in `seedBouncerStencils` and update that helper's doc comment, which names the seeded stencils.
Revise the ImpactSummary to say that a rotation which moves files now requires the stencil in the told stencils directory, and that the one test fixture which seeds a plan (`loomrecipe`'s `buildSequenceFixture`) is updated to seed it.

### [LOW] Card 1's ImpactSummary calls a refactor of `buildSettings` additive

**Location:** `_lyx/plan/01-claudeengine-deny-notice.md` (Intent and ImpactSummary)

**Issue:**
The Intent requires the Agent and AskUserQuestion install conditions to live "in one place both functions read, rather than restating them".
Today those conditions are inline in `buildSettings` (`cfg.ClaudeDenyAgentTool`, and `!interactive && cfg.ClaudeDenyAskUserQuestion` in the `else if` after the interactive marker branch), so meeting that requirement means rewriting `buildSettings`' body to read a shared helper.
The ImpactSummary opens with "additive: new unexported constants and one new function", which misdescribes that refactor.
Its conclusion that `buildSettings`' output is unchanged is correct as a goal, but the card never says what proves it.
`settings.go` being a file-level Edit target covers the edit itself, so this is a description defect only.

**Fix:**
Name the refactor in the Intent: `buildSettings` reads the same install-condition helper as `buildDenyNotice`.
Rewrite the ImpactSummary to state that `buildSettings`' body is reshaped around that helper, its output is byte-identical, and the existing `settings_test.go` cases pin that, with no caller outside the package.

### [NIT] Card 5 lists `composePlanPrompt` under Uses without depending on it

**Location:** `_lyx/plan/05-loomengine-prior-plan-block.md` (Uses)

**Issue:**
`Uses` feeds the derived dependency edges.
`PriorPlanBlock` is placed beside `composePlanPrompt` but neither calls it nor relies on its shape: it calls `stencilstore.Read` and `stencil.Fill` directly.
The entry records a location, not a dependency.

**Fix:**
Drop `internal/loomengine#composePlanPrompt` from card 5's Uses; the Intent already explains the placement.

## Focus directive verdicts

- **Granularity across cards 3 and 6:** not a defect.
  Card 3 changes the preparation seam's type, and `shedrecipe`'s Plan-Write wiring passes `NewPlanDirRotator`'s closure straight into `NewSingleLLMProducerGated`, so card 3 must reshape the closure or the build breaks.
  Card 3 carries only the shape change (always-empty amendment) and card 6 carries the content, so each card is independently reviewable and testable: card 3 against the producer's append behaviour, card 6 against the rotation's archive naming.
- **Decision-record fidelity:** every Decision (D1 to D7) and every Constraint is carried.
  D5's "embedded and seeded" is carried by card 4's `//go:embed` var and `entries` row, since seeding (`stencilstore.Reconcile`) iterates the registry.
  The stencil-enumeration Constraint is carried by `contracts/stencils/registry_test.go`'s registry-versus-disk test.
  Appending after `loom-rubric-webster-review` keeps indices 0, 4 and 5, which `internal/stencilcli/cli_integration_test.go` indexes, unchanged.
  The shell-quoting acceptance criterion is carried by card 2: `internal/shell` has exactly two implementations, posix and pwsh, and card 2 tests both.
  No card adds scope beyond the record, and no `manifest/designs/` doc describes the deny hooks or the preparation seam.
- **ImpactSummary accuracy:** card 2's claim holds, since `buildLaunchCmd`/`buildResumeCmd` are called only from `Claude.Prepare` (`claudeengine.go:104-105`) and `command_test.go`.
  Card 3's claim holds: every other `NewSingleLLMProducer`/`NewSingleLLMProducerGated` caller, including `internal/loomcli/smoke_attachprobe_test.go`, passes `nil`, and the two non-nil test closures in `singlellm_test.go` are in card 3's own targets.
  Card 6's claim holds for construction, since `loomcli/wiring.go`, `loomrecipe`'s fixtures and `shedrecipe`'s `newTestEnv` all set `StencilsDir`, but it misses the runtime break recorded as F1.
- **Card 1's install-condition refactor:** the tests are adequate, since the iff tests parse `buildSettings`' own output, but the Intent and ImpactSummary under-describe the refactor; recorded as F2.
