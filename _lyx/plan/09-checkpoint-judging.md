# Card 9 — Checkpoint judging and the circling guard

**Edit:**
- `internal/shedadapters/bouncerprompt.go`
- `internal/shedadapters/bouncer.go`
- `contracts/stencils/bouncer/bouncer-template-judge.md`
- `internal/shedrecipe/recipe.go`
- `internal/shedrecipe/entries_bouncer.go`
- `internal/loomcli/wiring.go`
- `internal/shedadapters/doc.go`
- `contracts/stencils/claims_test.go`
- `internal/shedadapters/bouncerprompt_test.go`
- `internal/shedadapters/bouncer_circling_test.go`
- `internal/shedadapters/bouncer_fixture_test.go`
- `internal/shedrecipe/entries_bouncer_test.go`
- `internal/shedrecipe/fixture_test.go`

**Uses:**
- `plan:internal/shedadapters#circlingEvidence`
- `internal/loomengine/config.go`

**Intent:**
D1 and D2: the judge may rule `CIRCLING` only from the configured checkpoint round on, and only on no-progress evidence Go can see.

Configuration path:
- `BouncerConfig` gains `CirclingCheckpoint int`, required positive by `NewBouncer`.
- `shedrecipe.Env` gains `ReviewCirclingCheckpoint int`, run-wide, which the Bouncer entry requires positive and passes through; `loomcli`'s `wire` fills it from `loomCfg.ReviewCirclingCheckpoint`.
- Test fixtures that build a `BouncerConfig` or an `Env` set it.

Prompt (D1):
- `bouncerprompt.go` gains a function returning the `decision_rule` marker for a round and checkpoint, held in Go so the stencil stays conditional-free, as `focusSchemaMarkers` already does.
  Below the checkpoint the rule offers `CONVERGED` and `CONTINUE` only and names no `CIRCLING`.
  At or above it, it adds `CIRCLING`: a gating finding on a key the facts list as open in an earlier round, unchanged or reopened, that is still open in this round; it states that a rising count or new keys alone is never circling, and that with progress the verdict is `CONTINUE`.
- The judge stencil's `## Decision rule` bullets are replaced by `{{.decision_rule}}`, the verdict vocabulary rule says `CIRCLING` is legal only where the decision rule offers it, and the header comment names the new marker.
- `judgeCall` fills the marker.
- `claims_test.go`: drop the "gating counts flat or rising" claim and keep the reopened-keys claim only where it still appears; add a claim that the stencil carries `{{.decision_rule}}`.

Guard (D2), in `settle`:
- A `CIRCLING` verdict with no decision file recorded for its round is read as `CONTINUE`, with a `logger.Warn` naming the producer, the round and the reason, when the round is below `CirclingCheckpoint`, or when `circlingEvidence(RunDir, round)` returns no key.
- Otherwise it proceeds exactly as a `CIRCLING` does today.
- Bound: the guard only narrows `CIRCLING` to `CONTINUE`; it never produces `CIRCLING` or `CONVERGED`, never touches a `CONTINUE` or `CONVERGED` verdict, and leaves a round that already has a decision file, and a legacy `APPROVED`/`BLOCKING` reading, as they are.
  It reads only on-disk state, so a harvest and a later replay of the same round agree.
- `doc.go` describes checkpoint judging and the guard where it describes the `CIRCLING` verdict.

Tests: `bouncerprompt_test.go` pins that the rendered judge prompt below the checkpoint contains no `CIRCLING` and at and above it does;
`bouncer_circling_test.go` covers a `CIRCLING` below the checkpoint harvested as `CONTINUE` (a `Stuck` with the ledger pointer), a `CIRCLING` at the checkpoint with no recurring open key harvested as `CONTINUE`, one with a gating finding open in rounds 1 and 3 reaching the existing `Awaiting`, a non-gating recurring key alone harvested as `CONTINUE`, and a rising count over new keys at the checkpoint harvested as `CONTINUE`;
`entries_bouncer_test.go` covers the new required field.

**ImpactSummary:** Every `BouncerConfig` and loom `Env` gains a required field; the judge prompt's decision rule moves into Go-held variants; an unearned `CIRCLING` no longer halts a run.

**Commit:** 9: Checkpoint judging and a Go guard on CIRCLING
