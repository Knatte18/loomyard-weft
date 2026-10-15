---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:87
    summary: D8 hands the re-measurement to a message no stencil or gate step ever makes the discussion writer send, so the carrier has no scheduled sender
    fix: >-
      Rewrite D8 so its carrier is filed by a step that is scheduled to run.
      Recommended: the carrier is a board note filed by the discussion writer itself with `lyx board upsert` (kind note, label audit, a slug of its own, body naming this task's slug), recorded in support-log.md's Question ledger as filed with the note's slug;
      the board's run lock refuses scope edits to a held entry but takes a new note (internal/boardengine/runlock.go:52), and the writer's Step 1 already runs `lyx board` commands.
      Drop the request to `ly:orch` from D8, the Rationale and Q9.
      If the author judges that the discussion stencil's write surface forbids a board write, say so in Q9 and choose another carrier whose sender is scheduled, instead of keeping a message nobody is told to send.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:87
    summary: D8 adds a writer-to-orch routing edge with no stated bound, no fallback when nothing is filed and no trigger on the note
    fix: >-
      In D8, state what the carrier can lose and what bounds it.
      Name the fallback for a run with no recorded parent, where the parent directive reads write your report and stop, and for an orch that files nothing.
      Give the note a trigger the operator can act on, for example the first Webster runs made after the deploy that carries this task, and the audit label that a re-measurement note such as `suite-weight-quiet-rerun` carries, so triage can tell when it is due.
      State in Q9 that a board note is passive: the orch only proposes and the operator promotes, so the note reaches an actor at triage, not at landing.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md:79
    summary: D8 and its Rationale state the note as filed at once, while Q9 records that the request is not yet sent
    fix: >-
      Make the three places agree on one state once F1 is settled.
      D8's Decision and Rationale describe the carrier as it will exist, not as an event that has happened, and Q9 records the real state (filed with slug, or still to be filed and by whom).
      Remove the sentence in Q9 that blames the round-2 fixer's messaging limit once the carrier no longer needs a message.
---

### [MEDIUM] D8's carrier has no scheduled sender

**Location:** `_lyx/discussion/decision-record.md:87`, `_lyx/discussion/support-log.md:78-79`

**Issue:**
The focus asks whether the carrier reaches an actor who will act on it when `alignment_pr_approve` is true and the landing runs no orch turn.
The board note itself does: it is a separate entry of kind note, it is not pruned with the done entry (`suite-weight-quiet-rerun`, an audit note from 2026-10-10, is still on the board), and the orch stencil's Board section already says a proposal goes on the board as a note at once.
The orch's After landing steps never read the decision record, so no landing-time carrier would work, and D8 is right to avoid one.

The weak link is the step before the note: the discussion writer's message to `ly:orch`.
Nothing makes the writer send it.
The discussion stencil's Steps 1 to 6 contain no such message, and the three gate messages it lists are a delivery prompt, a parent-review re-prompt and a keep-alive.
The delivery prompt tells the writer to send one fixed message to the reviewer and to follow it as written (`contracts/stencils/loom/loom-template-parent-review-delivery.md`).
D8 is written into the record, which the writer finished before the Burler loop began, and the writer is never told to reread it.
Q9 itself says the request is not yet sent.
The stencil's read-only rule is not the obstacle, since it governs files and not messages; the missing trigger is.

The parent-review route does not rescue it: the orch's fork reads the record only for a scope check, and an alignment agent may settle the review so that no orch fork reads it at all.
If the message is never sent, D8 reduces to nothing and the record still reads as if the carrier exists.

### [MEDIUM] The writer-to-orch edge has no stated bound

**Location:** `_lyx/discussion/decision-record.md:87-92`

**Issue:**
The record's own compaction rule, and the rubric's attack-surface question, require a Decision that adds a routing edge to say what it can skip and what bounds it.
D8 adds the edge writer to `ly:orch`, a direction the orch stencil does not cover: its Parent-review section only forbids messages toward the writer, and its Escalations section lists the driver, webster and a review loop.
D8 does not say what happens when the run has no recorded parent (the none variant of the parent directive tells the writer to write its report and stop), when the orch ignores or misfiles the request, or what the note must say to be actionable.
The note is passive: the orch proposes and the operator promotes, so a note with no trigger or label is read at triage whenever that happens.
The Rationale, the Decision and Q9 do name the same carrier, the board note by way of the orch, so they agree on it; they agree on a carrier whose bound is missing.

### [LOW] D8 and Q9 disagree on whether the note exists

**Location:** `_lyx/discussion/support-log.md:79`

**Issue:**
D8's Rationale says the orch files the proposal at once as a fact, and its Decision says the writer asks during this run.
Q9 says the request is not yet sent and explains why the round-2 fixer did not send it.
A reader of the record alone, which is all the plan writer sees, cannot tell that nothing has been filed.
