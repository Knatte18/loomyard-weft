# Card 22 — shedarbiter-package

**Create:**
- `internal/shedarbiter/doc.go`
- `plan:internal/shedarbiter#Deps` -> `type Deps struct`
- `plan:internal/shedarbiter#Input` -> `type Input struct`
- `plan:internal/shedarbiter#PromptInput` -> `type PromptInput struct`
- `plan:internal/shedarbiter#Call` -> `type Call struct`
- `plan:internal/shedarbiter#NewCall` -> `func NewCall(deps Deps, in Input) (*Call, error)`
- `plan:internal/shedarbiter#Call.Settle` -> `func (c *Call) Settle() (loomengine.ArbiterEnvelope, error)`
- `plan:internal/shedarbiter#Prompt` -> `func Prompt(stencilsDir string, in PromptInput) (string, error)`
- `plan:internal/shedarbiter#Spec` -> `func Spec(prompt, verdictPath string, settings sheddriver.ArbiterSettings) shuttleengine.Spec`
- `plan:internal/shedarbiter#VerdictGate` -> `func VerdictGate(verdictPath string) shuttleengine.GateSpec`
- `plan:internal/shedarbiter#ConsumeAttempt` -> `func ConsumeAttempt(attemptDir string, base shedengine.RelaunchRequest) error`
- `plan:internal/shedarbiter#Seam` -> `func Seam(deps Deps, runID string, escalationEligible func(producer string) bool) *sheddriver.ArbiterSeam`
- `plan:internal/shedarbiter#TestCall_SpawnPath` -> `func TestCall_SpawnPath(t *testing.T)`
- `plan:internal/shedarbiter#TestCall_Rerun` -> `func TestCall_Rerun(t *testing.T)`
- `plan:internal/shedarbiter#TestCall_VerdictOutcomes` -> `func TestCall_VerdictOutcomes(t *testing.T)`
- `plan:internal/shedarbiter#TestVerdictGate` -> `func TestVerdictGate(t *testing.T)`
- `plan:internal/shedarbiter#TestPrompt` -> `func TestPrompt(t *testing.T)`
- `plan:internal/shedarbiter#TestSpec` -> `func TestSpec(t *testing.T)`
- `plan:internal/shedarbiter#TestConsumeAttempt` -> `func TestConsumeAttempt(t *testing.T)`
- `plan:internal/shedarbiter#TestToldGeometryInvariant_AllowlistOnly` -> `func TestToldGeometryInvariant_AllowlistOnly(t *testing.T)`
- `plan:internal/loomcli#arbiterRecipeDescription` -> `func arbiterRecipeDescription(recipe string) string`

**Edit:**
- `contracts/stencils/loom/loom-template-arbiter.md`
- `internal/loomcli#arbiterPromptInput`
- `internal/loomcli#arbiterPrompt`
- `internal/loomcli#loomCLI.arbiterDepsFor`
- `internal/loomcli#TestArbiterPrompt`
- `pattern/PATTERN-cwd-resolution.md`

**Uses:**
- `plan:internal/sheddriver#ArbiterSettings`

**Intent:**
Move the arbiter's spawn and prompt out of `internal/loomcli` into `internal/shedarbiter`, a shed-layer, told-geometry package that batten imports without linking tree-sitter, beside the tackler in shed (D8).
The package carries `loomcli`'s arbiter call whole, exported: `Deps` (the status read, `sheddriver.ArbiterSettings`, the arbiter directory, attempt directory and marker path, the producers and goto guard, the eligibility readers `arbiterstop.Readers`, the pending-circling-decision read, the prompt's fixed inputs and stencils dir, the amendment target, the shuttle, the clock and the relaunch run id), `Input`, `NewCall` and `Call.Settle` with every branch of `settleExisting`, `attachAndConclude`, `settleFresh`, `concludeWait`, `conclude`, `resumePrecondition`, `handOn` and `finish`, the prompt (`Prompt`, `PromptInput`), the launch spec (`Spec`), the verdict gate (`VerdictGate`) and the resume consumption (`ConsumeAttempt`), each with the behaviour and texts `loomcli` has today, the records staying in `loomengine`.
`Deps` gains `HandBackWayForward`, the clause every refusal of the call ends with; an empty one takes `arbiterstop.HandBackWayForward`, today's loom text, so a recipe whose stop is read with another verb names that verb.
`PromptInput` gains `Recipe`, the recipe's description the stencil names under `## The run` through the new `{{.recipe}}` marker, as the tackler's prompt does: the loom and darn descriptions keep today's verbs (the circling verbs and `lyx loom decision add`), and a recipe whose description names no repair verb leaves the arbiter a plain re-launch, a goto or a hand-on; an empty decision record or support log path reads `none`.
`Seam` builds the `sheddriver.ArbiterSeam` a recipe's driver deps take: `EscalationPending` answers `escalationEligible`, `Judge` runs `NewCall` and `Settle` and maps the envelope onto `ArbiterOutcome` with role `loomengine.ArbiterRole`, and `AttemptEnvelope` reads the attempt record.
Until the next card switches `loomcli` over, its own `arbiterPrompt` fills the marker too: `arbiterPromptInput` gains `Recipe`, and `arbiterDepsFor` sets it from `arbiterRecipeDescription(recipe)`, the loom, darn and beam descriptions that `shedarbiter`'s callers pass from then on; `TestArbiterPrompt` gains the marker row.
The stencil's leading comment lists `{{.recipe}}`, and its body's "What you may change" section refers to the run's description for the verbs it may run.
The doc states the package's place: the arbiter call every shed-driven recipe's Go driver uses, told every path, importing no cwd resolver and no CLI package.
`pattern/PATTERN-cwd-resolution.md`'s "Bound packages" list gains `shedarbiter`.
Tests (tier0, parallel unless they capture logs): the arbiter tests of `loomcli` (`TestArbiterVerb_SpawnPath`, `TestArbiterVerb_Rerun`, `TestArbiterVerb_VerdictOutcomes`, `TestArbiterVerb_VerdictGate`, `TestArbiterPrompt`, `TestArbiterSpec`, `TestConsumeArbiterAttempt`) are carried over as `TestCall_SpawnPath`, `TestCall_Rerun`, `TestCall_VerdictOutcomes`, `TestVerdictGate`, `TestPrompt`, `TestSpec` and `TestConsumeAttempt` over a fake shuttle and clock, and `TestPrompt` gains rows asserting the recipe marker filled with a batten description and `none` for absent record paths; `TestCall_SpawnPath` gains a row whose `Deps.HandBackWayForward` is set and whose off arbiter's refusal ends with it; `loomcli`'s copies go on the next card.
`TestToldGeometryInvariant_AllowlistOnly` (tier0, in `seam_enforcement_test.go`, as `internal/sheddriver`'s is) pins the package's production imports to an allowlist with `internal/lyxcwd` and every CLI package denied.

**ImpactSummary:** A new package beside loomcli's still-live copy; the stencil gains one Go-filled marker, which loomcli's own prompt must fill until the next card switches it over.
