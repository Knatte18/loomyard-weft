---
round: 1
focus:
  - "contracts/stencils/seat/seat-directive-assistant-fanned.md and burler-template-assist-reviewer-fanned.md: does the composed fanned prompt still contradict itself anywhere (work alone, write surface, output_files, 'the one exception' in burler-assist-cluster-fan.md)? Settle whether 'work alone' now clearly means no other session, not no subagents."
  - "internal/burlerengine/assist.go, internal/seatengine/table.go, assistant.go, prompt.go, contracts/stencils/stencils.go: is the fanned stencil and directive chosen only when the seat is fanned, and does the unfanned path stay byte-identical? Settle whether every lens report is named on the write surface and in output_files, and whether keeping the reports off Outputs lets a run with an unwritten report still end."
  - "internal/shedadapters/roundusage.go, burlerreview.go and internal/lensreport/lensreport.go: does round-<N>-usage.yaml record the assistant's ending and each lens of a done fanned assistant with a missing file or a no-report outcome? Settle the failure paths: unreadable or malformed lens report, a non-done assistant, an unfanned seat, and whether lensreport is the format's one Go reader as pattern/PATTERN-sole-parsers.md now declares."
  - "pattern/PATTERN-sole-parsers.md and contracts/specs/twill-lens-report-spec.md: does the PATTERN change match the code that lands, and do the docs the repository's rules name for this kind of change land in the same diff?"
  - "Tests in assist_test.go, lensreport_test.go, prompt_test.go, table_test.go, burlerreview_assist_test.go and markers_test.go: test economy. Is any test duplicated by another, a per-helper test covered by a public-surface test, or a prose assertion on a stencil? Does any added behavior lack coverage?"
  - "Commit messages and _lym/landing/summary.md, when present: does each claim hold against the diff, and does any Pattern-Departure trailer name an entry that forbids the departure?"
---

Seed focus for round 1. The decision record carries only the board entry under `## Goal`, so the measuring stick is its Idea: a fanned seat's directive and write surface say the subagents are not other sessions and name the lens reports, and a round whose fanned assistant returned no lens report records that.
