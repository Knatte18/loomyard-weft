---
round: 1
focus:
  - "D2 and D3 in decision-record.md: for each new deny, does the Decision state what it lets through and what bounds it, and do the stated false-deny cases hold against the patterns as described (flag clusters holding `i`, `rm` with `$` elsewhere in the command)?"
  - "D5 in decision-record.md: does the claim that the widened sed grammar admits only reads hold for every address and range form it admits, including a regex address with a closing `/` followed by `,` and a second address, and does each acceptance-criteria example agree with the stated grammar?"
  - "D6 in decision-record.md: check the latch rules for consistency against each other and the acceptance criteria (set, clear, pane-read retry, restart rebuild, separate persisted offset), and whether the bound on the edge is stated for the notice, the pane read and the persisted field."
  - "D1 against D3 and the Constraints: the `.scratch/` scratch place, its removal by literal path, the `.git/info/exclude` seeding and the `rm` deny must not contradict each other, and the directive wording must stay admitted by the `PATTERN-no-denied-recovery` marker."
  - "Completeness before leanness: any requirement or constraint that sits only in support-log.md (Interview, Question ledger) and that the plan writer needs must be in decision-record.md's Decisions or Constraints; relocation findings for deliberation narrative left in decision-record.md are equally legitimate."
  - "Auto-mode assumptions and Open risks: is each Claude Code behaviour a Decision depends on named as an assumption or risk, with what a failure would mean, and is any assumption stated as fact inside a Decision?"
---

Seed focus for round 1.
The entries name where to look and which question to settle; none caps severity or pre-states a verdict.
The `Do not flag` list was applied: no entry asks for rejected alternatives in `decision-record.md`, a "Notes for the plan writer" subsection, or call-site enumeration.
