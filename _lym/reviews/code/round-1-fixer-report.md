# Fixer report — round 1

## Changes

- F1 (fixed as proposed, `fbf78aab`): `TestEngine_EntryPoints_ReadEngineBlocksForTheGeometrysDefaultEngine` in `internal/seatengine/engine_test.go`.
  It drives `Run`, `Probe` and `Resume` over a table whose seat names no engine and whose stencil includes an engine block.
  With `DefaultEngine` `codex` or `claude`, `Run` starts the chair with that engine's block text and `Probe` and `Resume` succeed; with it empty, all three refuse.
- F2 (fixed as proposed, `562a09ba`): `TestEngine_HalvesRenderEngineBlocksForTheirOwnEngine` and the `seedEngineWords` helper in `internal/burlerengine/engine_test.go`.
  The geometry's `DefaultEngine` is `codex`, the review choice names `claude` and the fix choice names none.
  The review half's step file and seat prompt must carry the Claude text, the fix half's the Codex text.
  The seat stencils include the engine block too, so a `New` that drops `DefaultEngine` fails.
- F3 (fixed as proposed, `b4bf3fd3`): `TestComposePrompt_RendersEngineBlocksForTheHalfsEngine` in `internal/burlerengine/template_test.go` seeds and asserts the explore step too; its seeding moved to `seedEngineWords`.
- Self-check follow-up (uncommitted until the final commit below): both `engine_test.go` file headers now name the new tests.

Gate: `lym gate test ./internal/burlerengine ./internal/seatengine` passed at tier 0 before the forks ran.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW, `internal/burlerengine/engine_test.go` header omits the new test and helpers: fixed as proposed.
- fix-consistency, LOW, `internal/seatengine/engine_test.go` header omits the entry-point engine test: fixed as proposed, by extending the header.
- fix-proposal: no findings.
- fix-affected-set: no findings; it left unchecked the helper blocks that go through `fillInstruction`, which the review did not name.

## Disputed

None.

## Deferred

None.
