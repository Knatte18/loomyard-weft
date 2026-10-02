---
round: 1
focus:
  - "D2 (attack surface): `lyx webster validate` moves from terminal-only scoping to scoping every begun card out of the resolve pass, so it stops checking begun non-terminal cards it checks today. Settle whether D2 states what that lets through and what bounds it, as D1, D3 and D5 do."
  - "D1 bound: it relies on the producing card's done-checks at `record-batch`/`recover-batch` to catch a missing target. Settle whether that holds for a begun batch that never reaches a terminal record (abandoned, or the run ends or escalates first), and whether the bound says so."
  - "D3 bound vs the #330 acceptance criteria: the bound compares a begun card's recorded hash with 'the plan-file hash the run had adopted before the rewrite'. Settle whether a per-card hash and a per-file hash are comparable as worded, and whether that comparison, or the existing foreign-edit check, refuses a foreign edit to a begun card followed by a sanctioned rewrite, as the acceptance criterion requires."
  - "D5 bound (attack surface): a successful fix skips escalation and keeps the outcome done, and the post-fix triage cannot see a test the strand deleted, skipped or weakened. Settle whether D5's own bound covers that, or whether it is left to Open risks and the later review rows only."
  - "D5: when Go rejects the strand's commits (plan directory or `_lyx` touched, HEAD moved by anything else), settle whether the Decision states what happens to those commits on the task branch before the run escalates."
  - "Relocation: `## Auto-mode assumptions` restates D1, D3 and D5 and names the alternatives they were chosen over ('over the board's...', 'not an in-session fork or a Webster-Review route'). Apply the completeness-before-leanness test: settle which items carry a requirement not yet in a Decision (one attempt with no config key, recovery role and timeout, no migration) and which are rejected-alternative narrative already in `support-log.md`."
---

D4 and D6 introduce no new edge or weakened guard beyond what D5's bound covers, so they get no entry.
`## Notes for the plan writer` is optional and its call-site lists are out of this review's scope, so neither its presence nor its completeness is a focus.
