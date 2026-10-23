# Plan cards: headroom check, Extends label, amendments, failure trace

Four changes to the plan and its cards, so that faults a reviewer or the operator meets today are caught or owned inside the run.

## What changes and why

1. **Headroom check at the plan gate.**
   The gate projects the `PATTERN.md` overview from the cards' verbatim entry lines and refuses a plan whose growth crosses the headroom line, before any review round.
   It also refuses a declared `adds` or `changes` entry that has no verbatim line.
   Before, a reviewing agent had to measure this by hand, and could be left with an unsatisfiable finding.
   The refusals (`pattern-headroom`, `pattern-line-unprescribed`) name their way forward.
2. **`Extends` card label.**
   A card can declare that it only adds to a shared ref (a flag, a list line, a doc paragraph, a spec entry).
   Two `Extends` on one ref are no dependency edge; `Extends` against a real change still is.
   One pair rule in `internal/planparser` serves `internal/websterengine`'s batch-order check and `tools/plandag`.
   `tools/plandag` gains the `E` mark and `-additive-guess`, which recomputes the card DAG with additive refs guessed by path and name, to measure the critical-path drop.
   The plan-review rubric and the plan-writer stencil check and teach the label; an `Extends` on a ref the card rewrites is a finding.
   An `Extends` on a `_test.go` file is refused, as an `Edit` on one is.
3. **An in-run owner for plan amendments.**
   `lyx loom decision add --amends-plan` writes `_lyx/loom/amendment.yaml`, tracked content committed with the decision entry.
   The Plan-Review settle record gains an optional `round_started_at`, and the settled-segment guard admits the amendment, so a Plan-Review round restarts and its fixer applies the amendment to the approved plan.
   The arbiter reaches that route through a new `amendment_target` prompt input, filled from a new top-level recipe key (`Plan-Burler-Review` in `loom-recipe.yaml`).
   The orch stencil and Webster's plan-conflict recurrence way forward take the same route.
   Before, no role could carry out a parent decision that amends a card, and the run parked until the operator was asked.
4. **Verify failures traced to plan cards.**
   When Merge-In or Publish writes the Publish failure record, the new `internal/failuretrace` package extracts `path:line` locations from the failed log, maps them through quarry's `Enclose` at the verified head, and matches the member glyphs to the plan cards whose targets hold them.
   The record gains optional trace fields, and the Publish failure note names the cards for the fixer.
   Locations are capped at 20 per record.
   `go.mod` moves quarry from v0.2.1 to v0.3.0 for `Enclose`; the one sanctioned location-to-glyph call lives in `internal/planglyph` and is added to the `PATTERN-glyph-conversion-chokepoint` background file.

Closes #735
Closes #806

## Design decisions to know

- The overview projection in `internal/pattern` keeps that package's import set (`PATTERN-leaf-packages`).
- New record fields (Publish failure trace fields, `round_started_at`) are optional, and older records read as before (`PATTERN-tolerant-records`).
- The amendment reaches the plan only through a review round's fixer, never by a hand edit; `landingshed` receives the trace as a told seam wired in `internal/loomcli`.
- `PATTERN.md` is already past its headroom line, so no overview entry is added.
- Also included: main's tab-separator fix for the reed pane tail (`internal/reedengine`), taken onto this branch because tmux 3.4 printed the old separator literally and its tests failed here.
  The lines match main's, so Merge-In should merge it cleanly.

## To check by hand

- The trace often names no card: Go test failure lines name only a file's base name, a production-code failure reported only by a test's assertion traces to the test, members built by a retired PR-Rework generation trace to none, and an opaque identity such as a `go vet` tier's carries no package.
  The untraced list and failing tests stay in the note for those cases.
- The hub orch rule that allows a hand edit of a card now conflicts with the orch stencil; the orch retires it with `lyx orch rules` after landing.
- A loom profile without the Plan-Review perch (`speed-loom-profile`) would leave an amendment with no fix loop.
- Open findings from review that no fresh reviewer has seen:
  - `card19-arbiter-doc-go-stale`: the arbiter's `doc.go` may still describe the old prompt inputs.
  - `d7-answer-key-carry-test-done-and-renumbered-cards`: the Plan-Burler-Review answer-key instruction's carry test for done and renumbered cards.
  - `d9-plan-handle-targets-never-match`: plan-handle targets never match in the trace's card matching.
- Integration-suite and verify-gate history: the first verify attempt failed on `cmd/lyx.TestCLIForm_EveryCommand` (fixed by trimming `lyx loom decision add` help under the 2000-byte `Long` cap, `81788dfa`), and the second on `internal/reedcli` and `internal/reedengine` tests (fixed by the tab-separator commit `b9b92feb`).
  Recheck that both areas pass on the landed head.
