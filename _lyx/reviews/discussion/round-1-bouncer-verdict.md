---
verdict: APPROVED
rationale: "All ten round-1 findings (F1 baseline inside batch 1, F2 failure-set/tail source, F3-F7 LOW gaps, F8-F10 NITs) are fixed in the revised pair; D7 now uses the per-batch StartSHA that state.go persists, D10 defines failures as the head set plus first-run-only identities, and no new finding surfaced."
---

# Round 1 judgment — discussion

The round-1 report was sound and in scope.
It carries no `## Focus departures` section, so there is nothing to ratify or reject.
Every finding it raised is checked against the revised `decision-record.md` and `support-log.md` below.

## Findings, checked against the revised pair

- **F1 (BLOCKING), baseline inside batch 1 — fixed.**
  D7 now takes the baseline from the HEAD webster recorded before the first batch in execution order began, and says why card SHAs are not used.
  That value exists: `BatchState.StartSHA` in `internal/websterengine/state.go`, set in `beginbatch.go` and carried through `recover-batch`.
  An absent recorded start falls to D8, and D8 states that rule.
  A new acceptance criterion covers a two-commit batch 1.
  Support-log turn 7, Q7, and a new rejected alternative record the change.
- **F2 (MEDIUM), which run's failures and tails — fixed.**
  D10 defines `failures` as the head set plus first-run-only identities, with the tail taken from the rerun when the identity failed there.
  D11 reads the regression section and `StuckReason` from `failures`.
  An acceptance criterion covers a rerun-only failure.
- **F3 (LOW), Master `stuck` path — fixed.**
  D3 says D11's recording applies unchanged, and the acceptance criterion now requires the `summary.md` triage section.
- **F4 (LOW), D2 rationale — fixed.**
  The rationale now rests on summary validation and the run-exit audit, which matches `mapMasterDone`; support-log turn 4 matches.
- **F5 (LOW), `outcome.yaml` after demotion — fixed.**
  D2 says Go never rewrites it and names the durable records of a demotion; the rejected alternative no longer cites the contradiction.
- **F6 (LOW), D9 predicate — fixed.**
  D9 defines failure at a SHA as membership in that SHA's parsed set and states the asymmetry with D7; a matching open risk is added.
- **F7 (LOW), stale fork log — fixed.**
  D4 removes any prior log before the integration prompt is rendered.
- **F8 (NIT), duplicate auto-mode assumptions — fixed.**
  One line marks D1–D11 as self-picks; the parser-placement bullet, the only one with unique content, stays.
- **F9 (NIT), plan-time facts — fixed.**
  D11 names the note and its condition instead of a call expression, and the `ParseReport` decoder-option note is gone.
  The remaining Notes bullets name files and functions at module-boundary level, which the writer stencil allows.
- **F10 (NIT), warning count — fixed.**
  D11 calls for one warning per non-empty category.

## Fresh pass

No new finding.
One downstream risk was checked and does not apply: D11 appends a section to a `done` run's `summary.md` after `mapMasterDone` validated it, and `landingshed` re-parses that file as the PR description.
`summaryparser.Parse` checks only the leading `# <title>` heading and takes the rest as body, so an appended `##` section cannot fail it.

The support log's `## Review rounds` still reads `_No rounds yet._`.
The writer stencil assigns that section to the review gate, not to the discussion author, so it is not a finding against this pair.
