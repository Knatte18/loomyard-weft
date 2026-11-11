```yaml
lens: board-scope
outcome: reported
```

Task: board-scope lens review of the parent-reviews-decision-record decision record.

**Findings**

1. LOW: `## Open risks`, first bullet (peer `beam-orch-root-project`). The line only notes a text overlap in `contracts/orchrole/orch-role.md`, which this lens does not count as a conflict.
   - Evidence: the peer entry wants one rule in the orch role about a beam orch passing a placement wish to its parent, "most likely under `## Messaging` or `## Working rules`". D6 adds its own section for `[discussion-review request]`. The two touch different sections and different subjects.
   - Fix: drop the line, or keep it only as a note to the plan writer.
2. LOW: `## Open risks`, second bullet (D6, the fork that writes). This is a design choice the writer has already made in D6, not a question only the parent can settle. Open risks is where parent questions go for the perch to raise.
   - Evidence: the entry asks for "the orch can hand the review to a fork of itself, which keeps its knowledge of the run but keeps the review out of the orch's main thread". D6 does exactly that. Nothing on the board contradicts it.
   - Fix: move the trade-off to the support log's Rejected alternatives (it is already there) and drop the risk line.
3. LOW: D2, against the issue #87 goal "Plan starts only on approval". The record never says where it departs from that wording.
   - The record makes the parent's answer non-gating, so approval stays the Bouncer's. A parent that sends findings in the last budgeted round never gets to confirm them, and the soft-cap rule, auto-accept or circling accept can then end the perch.
   - The entry's own constraint allows a deadline lapse ("a deadline lets the run go on to Plan without the parent's review"). Ending the perch while parent findings are open is a different matter, and the record leaves it unstated.
   - Evidence: D2's bound names `discussion_auto_accept`, the hard cap and `circling accept`, but it does not tie them back to the entry's "Plan starts only on approval" or say this is a deliberate departure.
   - Fix: one sentence in D2 that names the departure and why. The fix-class findings are fixed in-round, and a spent budget escalates to the parent anyway, which keeps the parent's say.

**Goal points of the entry**
- Optional, and off per hub in `loom.yaml` and per task in `run_settings`, with a default and a task override: met (D2, D5, D1).
- Issue #87, the parent reviews the record before Plan, approves or sends findings, and its findings go back as a review round: met, as a seat in the Discussion-Review round (D2, D3, D4, D7).
- Issue #87, Plan starts only on approval: met through the Bouncer, with the LOW gap in finding 3.
- Issue #87, the parent's load stays small: opt-in (D2), a fork (D6) and a deadline that records the lapse (D5) are all met.
- Complements `pr_review_ask_parent`: met. The record leaves it unchanged, which the entry allows ("could replace it" is not a requirement).
- Issue #113, a parent fan: met (D2).
- Issue #113, an agent fan with a named model and effort: met by the existing assistant reviewer (D1).
- Issue #113, either or both, each its own fan: met. The parent and the assistant are separate seats.
- Issue #113, reviews in the same round as the lenses, with findings joining the round and Burler-Fix fixing them in that round, and no pause, extra round or parent decision: met (D2, D7).
- Plan and Code perches "could later": correctly out of scope. Nothing on the board asks for them now.
- The Discussion's open question, fan or step or both: settled as a fan (D2).

**Entries, peers and commits checked**
- `beam-orch-root-project`: no conflict. It shares `orch-role.md` text only, in a different section and on a different subject.
- `anchored-hub-test-gaps`: no conflict. It is fabricengine tests only.
- `testmap-selector-upkeep-rules`: no conflict. It has no changed paths and covers the gate's coverage selector.
- Notes `codex-assist-review-trial` (R1) and `advisor-model-trial` (R4): no conflict. Both use the assistant and advisor seats as they stand, and D1 changes neither.
- Parent commits: `lym loom peers` lists none.
- Decision notes: no `Decision for parent-reviews-decision-record:` note is on the board, and none was unreachable.

**Tool calls**
- `lym board get beam-orch-root-project`: my context had its changed paths from `lym loom peers` but not its body, which I needed to judge the `orch-role.md` overlap.
