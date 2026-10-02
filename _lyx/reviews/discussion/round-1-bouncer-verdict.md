---
verdict: APPROVED
rationale: "All eight round-1 findings are resolved in decision-record.md as it stands (D6 source-file rule, D7 live-round definition plus re-classification bound, D9/D10 keyed on D7's round, D14 record.json marker, D1 path, read-only archive, D12 rationale and Auto-mode assumptions trimmed); no new blocking gap found."
---

The round-1 report raised four MEDIUM, two LOW and two NIT findings, and the fixer applied every one.
I checked each fix against the current `decision-record.md`, not the fixer report's own description.

- **F1 (Prosa exemption on code files).**
  D6 now makes a `Prosa` target that is a whole-package self glyph, or a file self glyph with a source extension of the plan's `language:`, force `required`.
  This matches what `checkProsaSymbolTarget` (`internal/planparser/validate.go`) admits.
  Open risks names the residual (a `Prosa` card on a stencil or rubric) and its bound (Webster-Review over the diff, the operator's PR review).
  The classification acceptance criterion covers the `.go` and whole-package cases.
- **F2 (skip seam trusts an agent-writable record).**
  D7 re-classifies the live committed cards and skips only when the stored class, the live class and `first_card` all agree.
  It also states the bound: a forged or stale record can at most skip review of cards that are exempt anyway.
  An acceptance criterion pins the stale-record case.
- **F3 (D9/D10 key on the wrong round).**
  D7 defines the live generation's round once.
  D9 and D10 key on that round and fall back to the generation-0 form without one, and D15 excepts Plan-Write's `archive-*/` rotation.
  After a `goto Plan-Write`, the fresh plan's `first_card` of 1 matches no rework round, because every rework round's `first_card` is above 1, so the fallback applies as intended.
- **F4 (crash marker).**
  D14 names `record.json`, written last, as the completion marker, and a record-less highest round directory as the interrupted archive.
  D2 and D6 now defer to D14, so the timing of `record.json` is consistent.
  D10's `head_sha` is already a field of today's `roundRecord` (`internal/loomshed/prrework.go`), so the range start needs no new data.
- **F5–F8.**
  - D1's path reads `_lyx/reviews/`, which matches `LoomReviewsDirRel`.
  - The Fabric Git constraint makes the archive read-only to the rework session, and Open risks names the residual.
  - The refusal-audit history now sits in support-log Interview 9 and is gone from D12.
  - Each Auto-mode assumption now cites the Decision it rests on and states only the board interpretation behind it.

The report has no `## Focus departures` section, so there is nothing to ratify.

Non-blocking observation: support-log Interview 5–7 and Q5–Q7 still describe the pre-review picks ("all-`Prosa` is exempt", "the highest committed round").
`Plan-Write` never reads the support log, and Review rounds is the gate's own append, so this is history, not a defect the plan writer can act on.
