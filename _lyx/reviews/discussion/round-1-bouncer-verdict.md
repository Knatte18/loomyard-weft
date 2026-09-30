---
verdict: APPROVED
rationale: "All seven round-1 findings (F1 D7 remedy text, F2 marker layout and reverse mixed-binary risk, F3 history_length risk closed as D10, F4 D6 restatement cut, F5 Goal and D1 reach, F6 and F7 nits) are applied in decision-record.md and support-log.md, the fold key covers every HistoryEntry field but at, and no remaining gap, relocation or exclusion finding survives the rubric."
---

# Discussion-Review, round 1 — judge verdict

The round-1 report returned APPROVED with seven findings, one MEDIUM, four LOW and two NIT.
The fixer applied all seven, and I checked each against the current artifacts rather than against the fixer report.

- **F1 (MEDIUM), D7 remedy.**
  D7's first sub-bullet now names both remedies: `lyx loom start` replaces a driver that has ended, and a live driver is inspected in its pane.
  A rationale line says start is not a re-delivery because the resume removed the park marker.
  The Auto-mode assumption, the battenshed acceptance criterion, Q9 and Interview item 6 match.
- **F2 (LOW), marker layout.**
  D7 states the layout: the unchanged identity line, then the decimal length.
  `internal/battenshed/innerrun.go:282` compares the whole file (`string(acted) == identity`), so the reverse direction is real.
  Open risks' Mixed binaries entry now records it with its bounded consequence, and an acceptance criterion covers the written layout.
- **F3 (LOW), `history_length`.**
  The open risk became D10, which states the freeze and both `ly-drive` consequences and accepts them.
  Q10 is resolved.
- **F4 (LOW), Activity recomposition.**
  The Open risks entry and the `persistTransient` Notes line are gone.
  D6 and its acceptance criterion survive, so the cut passes the completeness-before-leanness test.
  `internal/shedengine/run.go:539`'s `appended := len(nextHistory) > len(cur.History)` confirms D6 names a real rule the plan writer must implement.
- **F5 (LOW), fold reach.**
  The Goal names the driver-exit grace wait, and D1's rationale says only the awaiting wait is unbounded.
- **F6 (NIT), Notes.**
  The `nowRFC3339` line and the duplicate docs line are gone; the remaining Notes are module-level pointers.
- **F7 (NIT), D7 rationale.**
  The no-masking argument rests on D1 never folding an Awaiting entry, and the second case reads "any other child history length".

I also checked D1's fold key against `HistoryEntry` in `internal/shedengine/status.go:51`.
The key is producer, output and `gate_attempts`, with outcome Stuck and `budget_exempt` true implied by the rule, which covers every field but `at`.
So folding loses nothing but per-poll timestamps, as D1 claims.

Nothing in either file is flagged on the rubric's Do-not-flag list, and the writer stencil's bounded-exploration rule is respected: the Notes carry no signatures or `file:line` citations.
The overlap between the Auto-mode assumption on forward compatibility and the Mixed binaries risk is minor and does not warrant a finding.

The report carries no `## Focus departures` section, so there is nothing to ratify.
