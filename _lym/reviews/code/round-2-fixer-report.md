# Round 2 fixer-report

## Changes

- **F1 (MEDIUM), fixed in part, one site deferred.**
  Commits `eb0d8404` and `266bdaa5`.
  Each of four launch paths now runs a codex row and asserts the started spec's `PromptEngine` and `stencilkit.RequireEditDirectiveOf(..., "codex", "claude")`:
  - `internal/shedarbiter/call_test.go`: a `TestCall_SpawnPath` row for `settleFresh`, and a `TestCall_Rerun` row asserting the attach spec's `PromptEngine` for `attachAndConclude`.
  - `internal/sheddriver/tackler_test.go`: a `TestTacklerCall_SpawnPath` row for `settleFresh`, and a paused re-attach row asserting the attach spec's `PromptEngine`.
  - `internal/dobbyengine/tidy_test.go`: a `TestCountLandingsAndTidy` subtest over the spec `spawnTidy` hands `Start`.
  - `internal/loomcli/start_driver_test.go`: a `TestStartLLMDriverArm` subtest whose driver choice resolves through a registry alias to a codex engine.
  - `internal/gatecli/cardfixer_test.go`: a `TestRunWithFixer` subtest over the kit's engine reaching the fixer's spec and prompt (consumer side only; see Deferred).
  The fakes derive their answer from the `ModelChoice` they are handed, so a call site that passes a literal or the wrong choice now fails:
  the arbiter and tackler rows set `Settings.ModelChoice.Engine`, the tidy fake maps a choice's `Engine` through a table and answers claude for any other, and the loom row sets `cfg.Driver` and `registry`.
- **F2 (LOW), fixed as proposed.** `_lym/landing/summary.md` second design bullet now names loom's driver starter, beam's board-tidy shuttle, `RunnerShuttle` and the gate card fixer kit with their files. `_lym` is not committed by this round.
- **F3 (NIT), fixed as proposed.** Commit `405639dd` renames the six `tacklerPrompt()` messages in `internal/sheddriver/tacklerprompt_test.go`.
  Self-check fork finding: commit `3e475517` also renames the same stale name in the header of `contracts/stencils/shed/shed-template-tackler.md`.

- **Gate repair.** The gate rejected the two `wantEngines` doc comments (`internal/shedarbiter/call_test.go`, `internal/sheddriver/tackler_test.go`) for a line over the length limit.
  Each now reads as two sentences on two lines.

Gate: `lym gate test` over the five touched packages, tier 0, failed 0, before the comment repair; after it, `lym gate test ./internal/shedarbiter ./internal/sheddriver` passed, 368 of 368.

## Self-check

- fix-code-correctness, LOW, fakes ignore the choice: fixed with the choice-derived fakes above (`266bdaa5`).
- fix-consistency, MEDIUM, F1 item 5 half covered: deferred, see Deferred.
- fix-consistency, LOW, `wantEngines` declared twice in two packages: no action; the two fixtures are separate types in packages that cannot import each other's test code.
- fix-proposal, MEDIUM, fakes ignore the choice: fixed, same change as the correctness LOW.
- fix-proposal, MEDIUM, F1 item 5 derivation unpinned: deferred, see Deferred.
- fix-proposal, LOW, stale name in the tackler stencil header: fixed (`3e475517`).
- fix-affected-set, MEDIUM, F1 item 5 derivation unpinned: deferred, see Deferred.
- fix-affected-set, bouncer seed and judge sites in `shedadapters/bouncer.go`: no change. The review names `shedadapters` as pinned by its codex-models row, and I did not re-verify it.

All four forks returned a report.

## Disputed

None.

## Deferred

- **F1 item 5, `internal/gatecli/cardfixer.go:199`, `engine: rig.engines.EngineName(model)`.**
  The derivation stays unpinned.
  `newCardFixer` needs a hub, a loaded gate config and a built `newStrandRig`.
  Only `detachfront_integration_test.go` reaches that, so a tier 0 test cannot call it, and the fix key's "the `newCardFixer` test" does not exist.
  The new subtest pins the consumer in `detachfixer.go:259-277` only.
  Closing the gap needs an integration-tier scenario with a codex fixer alias, which is outside this round's tier 0 fix.

## Rulings

None.
