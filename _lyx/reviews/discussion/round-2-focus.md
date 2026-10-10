---
round: 2
focus:
  - "Check how decision-record.md D2's table values `pattern_directive` and `friction_directive` behave when empty (PATTERN inactive, Tier 2 off, friction read failure) against `seatengine.Table.Validate`, which refuses a blank value in `internal/seatengine/table.go`, and `composePrompt`, which fills with `missingkey=error`: does the Decision state a working mechanism, and does an Acceptance criterion cover those cases?"
  - "Check D4's `discussion_advisors` shape against `ModelSpecList.UnmarshalYAML` in `internal/loomengine/config.go`, which loads an empty scalar as a one-entry list holding an empty string: do the Decision and the Acceptance criteria name how each spelling (empty scalar, null, empty list, a sequence holding an empty string) yields an advisor count?"
  - "For the advisor seat and its advisor-to-chair message edge (D5, D8), the discussion commit stages the whole `_lyx/discussion/` directory (`internal/loomcli/wiring.go`): does a Decision state what an advisor that ignores its brief can still do and what bounds it, rather than leaving the limits to stencil text alone?"
  - "Check D3's appended chair-prompt amendment against the Scope's In and Out lists: does the append require a seatengine change that Scope names, and is it justified by a requirement of this task rather than by a note Scope puts Out?"
  - "Check that each Decision introduced or changed in this round's fixes states what it can skip or let through and what bounds it, and that any new text keeps rejected-alternative narrative in support-log.md while the requirement it carries stays in decision-record.md."
---

The focus lists question areas only; the reviewer still reviews the whole pair.
