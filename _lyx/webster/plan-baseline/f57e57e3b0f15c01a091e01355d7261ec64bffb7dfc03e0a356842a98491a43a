# Card 11 — Discussion-Write reads the board and handles a parent review

**Prosa:**
- `contracts/stencils/loom/loom-template-discussion.md`

**Edit:**
- `contracts/stencils/discussiontemplate_test.go`

**Intent:**
Update the Discussion-Write stencil for decisions D5, D8 and D11.

Step 1 gains: after reading the task's own board entry, run `lyx board list` and `lyx board get` any entry that touches this task, and treat planned work as design context the design must fit without duplicating or contradicting it.

A new short section, placed after the self-check step, describes the two messages the gate may send after the writer ends its turn.
A delivery prompt asks it to notify the parent reviewer and record the delivery; it follows that prompt and changes nothing in the discussion.
A parent-review re-prompt names a review file; the writer addresses every finding, BLOCKING and NIT alike, by fixing it or by answering it, and appends one parent-review entry to the support log's `## Review rounds` listing each finding with its disposition (fixed, or answered with the reason), then re-runs `lyx loom validate-discussion` and ends its turn.

"What you may write" gains two carve-outs: running `lyx loom review delivered` when a delivery prompt asks, and appending the one parent-review entry to `## Review rounds` when a parent-review re-prompt asks.
Every other `## Review rounds` entry stays the Discussion-review gate's.

`discussiontemplate_test.go`'s fence tests accept the two carve-outs and assert the stencil names `lyx board list` and both carve-outs.

**ImpactSummary:** Prompt text for one producer plus its pinning tests; no marker is added or removed, so `composePrompt` is unchanged.
