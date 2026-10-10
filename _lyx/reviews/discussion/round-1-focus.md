---
round: 1
focus:
  - "D2 in decision-record.md: for each admitted form (quoted glob anywhere a positional is, unquoted glob after `--`, leading `~`, adjacent quoted and unquoted parts joined into one word), does the Decision state what it lets through and what bounds it? Check in particular that a quoted glob or a joined word cannot dequote into a flag-shaped or `--`-shaped word that escapes the flag-must-match rule."
  - "D3 in decision-record.md: the trailing `--json` rule admits any command words before it. Settle whether the bound (only `lyx shed status` declares a local `--json`, pinned by a test) holds for verbs that take a positional or flag whose value is `--json`, and whether `--help -h` ordering and bare `lyx --json` leave a path that runs a verb."
  - "D1 in decision-record.md: does the class-prefix predicate bound what it skips? Check that a real path whose text starts with an audit class name followed by `: ` cannot be mistaken for a pathless entry and thereby skip a genuine link check."
  - "D6 against Acceptance criteria and the Auto-mode assumptions: do the four dead-record cases cover every combination of record kind, recovery count and committed-work state, with no case left falling through to the old 'exhausted' text, and do the Decision and the acceptance rows agree on each case's wording?"
  - "Cross-section consistency: D4's list of rejection rows against the Acceptance criteria rows (for example the `ls-remote` row and the `branch` flag), and D5's single `way forward:` clause against the Acceptance criteria for the CLI path and the fresh partition."
  - "Relocation and exclusion candidates: Auto-mode assumptions, Open risks and Notes for the plan writer. Before proposing any move, check whether the content carries a requirement the plan writer needs and extract it into Decisions or Constraints first, since Plan-Write never reads support-log.md."
---

Seed focus for round 1.
Entries target the Decisions that add admitted forms or loosen a guard (D1 to D3), the multi-case refusal wording (D6), and cross-section agreement.
Per the rubric, absent rejected alternatives, absent plan-writer notes and incomplete call-site enumeration are not findings.
Per the writer stencil, exact signatures and file references the writer was told not to gather are not flagged as missing.
