```yaml
lens: goal-scope
outcome: reported
```

The goal-scope lens checks the decision record against task T2's board entry.

**Goal points of the entry:**
- Loom and darn merge a moved parent in Finalize itself: the conflict session resolves a conflict under the parent lock, and an overlapping merge goes on to the landing gate, whose fixer fixes what fails. Met by D1 and the Scope In list.
- A conflict or overlap in Finalize no longer reroutes, so the Code-Review perch is not re-run for it. Met by D1. Scope Out keeps `on_reroute` only for a spent fixer cap or an over-cap change, which the entry keeps as they are.
- Parent-Align stays the loom row before Code-Review that takes the bulk of the drift. Met by Scope Out.
- Pawl keeps its own full landing treatment. Met by the Constraints pawl line. D2 drops `LargeMergeLands`, but pawl's behaviour stays the same.
- A landing fixer's `escalate` goes to the arbiter first, and only the arbiter's hand-on reaches the orch. Met by D3 and D4.
- A spent cap, a failed or unusable fixer turn and an over-cap change stay as they are. Met by Scope Out and the last acceptance criterion.
- The escalate is "one more row for the seam, not a new mechanism", and the landing-halt rule row yields for it. Met by D3, plus D4's reuse of `JudgesFirst`. D4 widens the signature with the stop's reason, and support-log's Rejected alternatives justifies that from the entry's own wording. That is a choice the writer may make, not unlicensed work.
- Docs: the orch role text on landing halts changes. Met by D5's first bullet. D5's other doc edits are the same-commit doc rule from CLAUDE.md, not extra work.
- Unlicensed work or deferral: none found. Batten's re-step skip (D3) and the removal of darn's overlap note and its review input (D2) follow from the entry's design. D4's build hand-over re-step is a stated accepted bound, not a deferral.

**Decision notes:** `lym board list` shows no note titled `Decision for landing-settles-in-place:`, so there were none to read. The record has no `Added after Discussion` entries.

**Findings:** none.

Tool calls:
- `lym board list`, filtered to this task and any decision notes: my context held the entry body but not the board list, which this lens needs to find decision notes.
