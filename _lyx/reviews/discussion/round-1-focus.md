---
round: 1
focus:
  - "Attack-surface question, D3, D8, D9: each removes or skips something (a reflection session, a reload, a skills step) without a `Bound:` line.
    Does each state what it can now skip or let through and what bounds it, or is the bound only implied elsewhere (R3, Constraints)?"
  - "Attack-surface question, D1, D4, D5, D10, D11: each has a `Bound:` line.
    Does the bound actually limit the new edge?
    D4's phrase 'terminal outcome or is otherwise not attachable' needs checking against the attach probe's own notion of attachable, so that a run the probe would attach to is never removed.
    D1's input-box reading must be able to tell the sent text from a draft of other text."
  - "Premises, D9 and D10: the decisions rest on A1 (`/reload-plugins` exists, rebuilds the Skill tool's list and ends no turn) and A2 (skills survive a compaction, not `/clear`).
    Is any premise false or unsupported in the record, and does the record say what happens if it fails?
    R1 covers D10 only."
  - "Consistency, D8 against its acceptance criteria and R3: does the freshness rule ('no turn end between the boundary and the turn end that made the watcher look') mean the same as 'a compaction followed by exactly one turn end enters the reload sequence' and 'later turn ends moves the baseline'?
    Check that the criteria split the two cases at the same point as the decision."
  - "Consistency, D10 against D8 and D9: the order of `/reload-plugins`, skills step and pointer step across the three reload entries (after `/clear`, watcher-run compaction, auto-compaction), and the wait for the next idle probe after a command that ends no turn."
  - "Design, D6: the decision fixes an outcome and leaves the cause to the plan writer.
    Does it still give the plan writer a testable, bounded requirement, and does D6's bound ('a driver that dies mid-producer is still reported') agree with its acceptance criteria?"
  - "Scope, D7, D11 and D13: D7 widens the note's content, D11 adds a state field and a wait, D13 deletes attach paths.
    Does each trace to a bug in Scope, with no extension the Out-of-scope list forbids?"
  - "Relocation and exclusion: look for deliberation narrative inside `decision-record.md`'s Rationale lines that belongs in `support-log.md`, after the completeness-before-leanness test.
    Also look for content that does not belong in Discussion at all."
---

Seed focus for round 1.
The three `Do not flag` items were checked against these entries.
Notes for the plan writer are not an entry, rejected alternatives are not an entry, and the stencil's ban on signatures, `file:line` citations and exhaustive enumeration means no entry asks for them.
