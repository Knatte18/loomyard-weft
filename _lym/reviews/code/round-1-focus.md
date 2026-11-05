---
round: 1
focus:
  - "`internal/sheddriver/drive.go` `noticeArbiterSettled`, called after an arbiter `resume` at an awaiting stop: does the entry's 'the orch hears only the outcome' hold for every resume the arbiter can make there, including a resume whose goto is not a settled escalation, and does `HandBack.Notify` have side effects (parking, a stop report, a relayed notice) that contradict 'asks nothing'?"
  - "`contracts/stencils/loom/loom-template-arbiter.md`: does the new text keep the entry's limits — one edit the stop asks for and nothing wider, `--by arbiter` on every recorded call, hand-on only for a question the entry's design and the record's decisions leave open — and does it contradict any rule left unchanged elsewhere in the stencil, such as 'What you never do' or the amendment-target paragraph?"
  - "The arbiter stencil's `decision` rule: can the arbiter accept or continue at a `decision` escalation in a way the circling and hard-cap rules around it make impossible, such as a `continue` at a cause whose cap rules differ, and does the 'A `resume` at a review escalation needs its circling decision recorded first' rule still apply to the new cause?"
  - "Eligibility change in `internal/arbiterstop/arbiterstop.go`: does every other consumer of the eligible-cause set (the loom CLI's arbiter verb, batten's hold, the driver's rule table, the escalation brief and stencil text in `shedadapters`) agree that `decision` is now eligible and `budget` is not, or does one still treat a `decision` escalation as parent-only?"
  - "Docs and prose that describe who settles a `decision` escalation (`contracts/orchrole/orch-role.md`, `contracts/stencils/bouncer/bouncer-template-escalation.md`, the `doc.go` files, the `EscalationCause` comment): do they now agree with one another and with the code, and do the orch-role lines still tell the orch to act on an arbiter hand-on of a `decision` escalation?"
  - "Test coverage of the new behavior: does a test cover the notice's no-orch-recorded and failed-send paths, and does one cover the eligibility change end to end through `Check`, or is only the table row for `decision` pinned?"
---

Seed focus from the decision record's Goal entry, read against the diff stat and the two commits.
The record's other sections are empty, so the entry is the whole measuring stick.
The open questions it lists (how far the arbiter decides alone, whether `--by arbiter` is a new author) are for the diff to answer in the stencil and the decision-add path; check whether the diff touches the author validation for `--by arbiter`.
