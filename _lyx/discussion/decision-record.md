# Decision record — rewrite C13's stacked half, commit the creel re-test report

## Goal

Make C13 in `tools/sandbox/SANDBOX-CREEL-SUITE.md` describe a setup that exists under the window-per-strand layout, and commit the creel stage 1 re-test report as a measurement record.

## Scope

- Rewrite the stacked 3-row half of C13 (heading, Goal, Watch) in `tools/sandbox/SANDBOX-CREEL-SUITE.md`; the hand-added half stays as is.
- Copy `/home/knatte/Code/loomyard-LYXHUB/loomyard/.scratch/creel-retest-2026-10-17.md` verbatim to `docs/research/creel-audit-2026-10-17/report.md`.
- Both files land in one commit; no code changes.

## Decisions

- The second C13 strand is a `lyx reed add --cmd bash` strand in its own window, so creel cannot find an input box in its pane.
  Rationale: it provokes the `no-input-box` stall deterministically; a stacked pane can no longer be built, and `too-short` now arises only from a short viewer terminal, which a scenario cannot set up reliably.
- The scenario keeps its check that the stall reason shows as a `stalled` event in `lyx creel log` and as a progress line on stderr of `lyx creel inject --wait`.
  Rationale: the silent-stall `FAIL` bucket is the point of C13 and is independent of layout.
- The heading becomes "Hand-added and stalled strands".
  Rationale: "Collapsed" names the removed `collapsed_rows` layout.
- The report is copied byte for byte.
  Rationale: it is a measurement report of one run.

## Constraints

- Docs only; the tree's code and tests stay untouched.
- Markdown follows semantic line breaks.

## Auto-mode assumptions

- The report source file exists and is final at copy time.

## Open risks

- A stall other than `no-input-box` is not exercised by the rewritten half.

## Acceptance criteria

- C13 mentions no stacked layout, no 3-row pane and no collapsed strand.
- `diff` of the source report and `docs/research/creel-audit-2026-10-17/report.md` is empty.
- One commit holds both files.
