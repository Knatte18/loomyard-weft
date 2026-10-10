# Plan round 2 — fixer report

Every finding was validated against the code and fixed.
`lyx loom validate-plan` passes over the fixed plan (`ok: true`, informational `create-new-unit` findings only).

## Fixed

- **F1** (BLOCKING, scope) — confirmed: `TestFullEnv_FillsEverySeam` asserts `NilSeams(FullEnv(t))` is empty, battencli's `TestWire_EverySeamFilled` fails on any nil seam outside `intentionallyNil`, and `NilSeams` reports nil interfaces and maps, which covers `Seats` and `Models` (`modelspec.Registry` is a map).
  Card 10 now lists `internal/testkit/envkit#FullEnv` and `internal/battencli#intentionallyNil` under `Edit:`.
  Its Intent has `FullEnv` fill `Seats` with a `shedfake.SeatRunner` and `Models` with a small registry carrying the `opus` alias, and has batten list both as loom-only seams.
  The ImpactSummary is updated to match.
  No test is added, because the existing seam tests cover the fill.
- **F2** (BLOCKING, scope) — confirmed: `TestBuild_EveryRegisteredEngineBuilds` ranges `shedrecipe.Names()` over `engineMinimalConfig`, which has no `MultiLLM` key.
  Card 11 now lists `internal/shedbuild#engineMinimalConfig` under `Edit:`.
  Its Intent adds a `MultiLLM` entry: a role, a segment, a chair and one advisor on the stencil name the function already takes, the advisor's output among the chair's inputs, and the `opus` alias from card 10's `FullEnv`.
  The signature is kept, so no re-sign arrow is needed.
  The doc comment's engine count is updated.
- **F3** (MEDIUM, design) — confirmed: `FillOptional` parses with `missingkey=error`, and a bare `{{template "name"}}` executes the block with nil data.
  Card 2 now fixes one rule that keeps D3's bare form: `FillWith` sets each bare include action's pipeline to dot in the parsed tree before executing, and refuses an include action that carries its own pipeline, naming the block.
  `TestFill`'s planned rows gain the refusal row, and the rendering row now names the bare include explicitly.
  No registered stencil carries a `template` action (checked by grep over `contracts/stencils`), so `FillOptional`'s byte-identical claim holds.
  Card 3's header wording now names the bare form.
  Cards 5, 12 and 13 already use the bare form, so they need no change.
- **F4** (LOW, consistency) — confirmed: D11 asks for a test over `seatengine` itself, and only card 9's adapter test pinned N = 1.
  Card 6's `engine_test.go` now runs its chair-outcome rows over a one-chair table as well as a chair-and-advisor table.
  These are rows in the existing test, not a new test function.

## Disputed

None.

## Deferred

None.
