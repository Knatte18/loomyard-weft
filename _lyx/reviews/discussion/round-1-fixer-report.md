# Round 1 fixer report — discussion

All ten findings are fixed.
`lyx loom validate-discussion` exits 0 on the revised pair.

## Changes

- **F1 (BLOCKING), baseline.**
  D7 now defines the baseline as the HEAD webster recorded before the first batch in execution order began.
  It also says why card SHAs are not used, and that an absent recorded start makes the baseline unavailable.
  D8 applies its "baseline unavailable" rule to that case.
  A new AC covers a two-commit batch 1.
  Support log: interview turn 7, Q7, and a new rejected alternative for the old definition.
- **F2 (MEDIUM), failure set and tails.**
  D10 now defines `failures` as the head set plus first-run-only identities.
  Each tail comes from the rerun when the identity failed there, and from the first run otherwise.
  D11 says the regression summary section and `StuckReason` read from `failures`.
  A new AC covers a rerun-only failure.
- **F3 (LOW), Master stuck path.**
  D3 now says D11's recording applies unchanged, and the AC requires the `summary.md` triage section on that path.
- **F4 (LOW), D2 rationale.**
  The rationale is reworded around summary validation and the run-exit audit.
  Support-log interview turn 4 is updated to match.
- **F5 (LOW), outcome.yaml.**
  D2 states that Go never rewrites `outcome.yaml`, and names the durable records of a demotion.
  The rejected promotion alternative no longer cites the `outcome.yaml` contradiction.
- **F6 (LOW), D9 predicate.**
  D9 now defines what "fails" means at a SHA and states the asymmetry with D7.
  A new open risk covers mislocalization when a test did not run at an intermediate SHA.
- **F7 (LOW), stale log.**
  D4 now says webster removes any prior log before it renders the integration prompt.
- **F8 (NIT), auto-mode assumptions.**
  The five bullets that restated decisions are replaced by one line marking D1–D11 as self-picks.
  The parser-placement bullet is kept.
- **F9 (NIT), implementation facts.**
  D11 names the friction note and its condition instead of a call expression.
  The `ParseReport` decoder-option note is removed.
- **F10 (NIT), warning count.**
  D11 now calls for one warning per non-empty category.

## Deferred

None.
