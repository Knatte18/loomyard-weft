---
round: 1
focus:
  - "decision-record.md D6: replacing configreg's literal name list with sorted/unique/template-backed properties removes the check that notices a section being dropped; settle whether D6 states what can now slip through and what bounds it, as D5 does for deleted verbs."
  - "decision-record.md D9: replacing wantProducerTable's routing columns with a perch-shape property downgrades a guard; settle whether the stated OnDone-swap proof is a bound on what the property now lets through (for example a reciprocal Bouncer/Burler pair wired to the wrong segment's targets), or whether a Bound line is needed."
  - "decision-record.md D14 against D20 and D21: D14 deletes loomcli tests with no Bound line of its own; settle whether D20's Shed Verb-Set claim and D21's new-owner rule bound the deletion, including whether loomcli's own use of the shedverbs functions stays asserted somewhere."
  - "decision-record.md D18: the new Tmux Test Isolation scan carries an allowlist (entry plus reason); settle whether the Decision bounds what may go on that list, since an allowlist is an escape hatch under the rubric's attack-surface question."
  - "decision-record.md D19 and Constraints: the tmuxkit os/exec exemption is bounded to the tmux binary by review only; settle whether that bound is sufficient as stated, or whether the rebuilt Testkit scan is meant to enforce it mechanically and the record should say so."
  - "decision-record.md D17 and Auto-mode assumptions: the whole leak fix rests on production reed addressing tmux only by -L and on child processes inheriting TMUX_TMPDIR; settle whether those assumptions are stated as decisions the plan can check, and whether D18's 'untagged entry point spawns nothing' claim holds given the entry point runs in every tag set."
  - "decision-record.md Acceptance criteria, socket-directory criterion: settle whether 'excluded by checking which servers answer' can tell a leaked live test server from an operator hub server, so the criterion is checkable as written."
  - "decision-record.md D20's 'audited and unchanged' paragraph, the per-Decision Rationale lines and Notes for the plan writer: apply the completeness-before-leanness test; settle which parts carry a constraint the plan writer needs and which are audit or deliberation narrative that belongs in support-log.md, and whether any Notes line only restates a Decision."
---

The record is large (24 Decisions), so round 1 targets the rubric's two sharpest questions: attack-surface bounds on every removed or downgraded guard and every new escape hatch, and relocation of narrative out of the decision record.
D2, D5, D7, D10, D12, D13 and D17 already carry explicit Bound lines and are not seeded; the reviewer may still raise them.
Per the writer's stencil, missing signatures, `file:line` citations, call-site lists (for example D11's reliance on #327 §3 for the seeder list) and rejected alternatives in the decision record are out of bounds and not seeded.
