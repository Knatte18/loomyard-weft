---
verdict: APPROVED
rationale: "All five round-2 findings are fixed in the current plan: card 8 names a same-call fork fabric-reference on an OK report with a failing verify and card 11 recovers that case by name, card 1 compares link-resolved paths with a link-spelled test row, card 10 records later-card drift through recordBatchWarning into state.json and the summary, and cards 27 and 23 fix their pointers; the fixer's edits introduce no new finding"
---

# Round 2 verdict — APPROVED

## Round-2 findings: all resolved

The review was written against the pre-fix plan.
The fixer's report lists a change per finding, and each change is present in the current cards.

- F1 (BLOCKING, carried from round 1): `08-record-batch-audit-dispositions.md` now has the bullet "A fork `fabric-reference` on an `OK` report whose card verify exits 1, on the same call (step 7's first trigger)", asserting the archive, `errors.Is(err, ErrBatchFailed)` and the `recover-batch` message.
  `11-recover-failed-batch.md`'s recovery test names "card 8's same-call `fabric-reference` on an `OK` report with a failing card verify".
  D5's must-cover scenario now has an unambiguous test, and deleting step 7's verify re-run would fail it.
- F2 (MEDIUM): `01-audit-severity.md`'s `ClassifyViolation` resolves the write path and each told directory through links before every prefix test, and the test table adds a row with `_lyx` made a link through `fslink.CreateDirLink` and a `parent-write` spelled through the target, expecting correctness.
  That matches D4's "anywhere under the run's `_lyx` content".
  The Told-Geometry reasoning holds: resolving a told path's links derives no new segment.
- F3 (MEDIUM): `10-post-batch-failures.md` records each later-card drift finding with `recordBatchWarning` under `later-card-drift:<finding text>`, through a new `postBatchInputs.Batch` both callers fill.
  Its drift test asserts the entry in batch 1's `AuditWarnings`, and its ImpactSummary says so.
  `02-audit-ledger.md`'s `recordBatchWarning`/`recordRunWarning` now take `class` and `detail` strings.
  Card 8 and card 15 name those helpers without fixing their argument lists, so they are still consistent.
  `14-summary-audit-warnings.md`'s explanation covers drift warnings as well.
  This carries D9's "D5's recording path".
- F4 (NIT): `27-loom-gate-refusal-sweep.md` limits the card 25 pointer to `seed-missing` and the decode and invalid-state branches, and says the done state gets no row because step 1's short-circuit is not a refusal.
- F5 (NIT): `23-websterengine-refusal-sweep.md` names the overview's `way-forward-phrasing` shared decision instead of a card range.

I ratify all five as legitimate under the rubric.
F1 concerns must-cover verify coverage, F2 and F3 concern fidelity to D4 and D9, and F4 and F5 concern cross-reference accuracy.
None of them re-derives a gate check or a dependency edge.
The review carries no `## Focus departures` section.
Both round-2 focus entries were addressed: the first by F1, and the second by the review's own cross-reference sweep plus F4 and F5.

## Checked without a finding

- The fixer's edits did not shift any antecedent.
  Card 8's bullets each name their finding.
  Card 11's three recovery sources each point at a named case.
  Card 10's `later card:` wording and its `later-card-drift` class describe the same recorded warning.
- Card 1's canonicalization uses `filepath.EvalSymlinks` on the nearest existing ancestor.
  That is the approach `internal/fabricengine/ancestors.go`'s `resolveAncestorSymlinks` already takes over the same warp/weft junctions, so it is not a new cross-platform risk.
  The card's link-spelled test row runs on every platform the integration suite runs on.
- Decisions D1–D12 and the listed constraints still map to cards, and the new drift recording falls within D9.
- `## verify:` is unchanged and still covers every targeted package, with `integration` run and `smoke` compiled through `go vet`.
- No `Custom` card exists.
