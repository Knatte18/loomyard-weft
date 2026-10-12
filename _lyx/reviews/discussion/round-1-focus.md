---
round: 1
focus:
  - "Decision 4 (whole-file rule dropped): does its stated bound cover what the removed guard let through, including a self-fix or a card that edits a file another card's listed span covers? Settle whether the claim 'batches run one fork at a time and no edit has happened yet' is a premise the record states or relies on without support."
  - "Decision 5 and Decision 4 together: with the member-name rule gone and only one sentence left as a stale-offset guard, is anything a fork could still do with a stale listed line unbounded, or does the Edit tool's text matching plus the gates bound it as claimed?"
  - "Decision 2 (batch-wide reads of uncovered files and unresolved glyphs before any edit): it widens the board entry and adds a read-before-edit edge; does it state what it can now skip or let through, and is the 'plan: handle is never located' rule consistent with the acceptance criteria for a card that creates a member?"
  - "Relocation and exclusion: do Auto-mode assumptions and Open risks restate Decisions 2 and 4 rather than add a constraint, and does anything in decision-record.md carry only deliberation narrative that belongs in support-log.md? Apply the completeness-before-leanness test before suggesting any move, since Plan-Write never reads support-log.md."
  - "Acceptance criteria against Decisions: is every Decision (1 to 5) covered by a criterion the plan writer can check, and does any criterion assert something no Decision decides?"
---

Rationale: the design is a wording change inside existing stencil fragments, so the risk sits in the guard removed by Decision 4, the batch-wide edge added by Decision 2, and record leanness.
The "Notes for the plan writer" subsection, rejected alternatives and call-site enumeration are by design out of the rubric's flagging scope and are not focus items.
