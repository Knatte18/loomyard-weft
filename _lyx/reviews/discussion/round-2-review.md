---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D7 (third bullet) and Acceptance criteria (last bullet)
    summary: D7 hands the dropped #673 entry to a Merge-In resolution that never runs on a clean merge, so no actor owns the case where the parent loses the workaround first
    fix: In D7's third bullet, replace "and the Merge-In resolution drops it" with an owner that runs on every merge-in, for example "the Merge-Review perch checks, after each merge-in, whether the parent's CLAUDE.md still holds the #673 workaround, and if it does not, the task's ORCHESTRATOR.md entry for #673 is a finding fixed by removing it". Name that owner in the matching Acceptance criterion too. In Open risks, turn the board-note follow-up into a Decision or a named action, since the plan writer schedules only Decisions, Scope and Acceptance criteria.
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D1 (first bullet, brief settle condition and Bound)
    summary: D1's bound rests on a recorded operator decision but lists sources that also hold arbiter, parent and auto-mode decisions, so the fork can settle on an agent's own call
    fix: In D1, define a recorded operator decision as the board entry and the operator's board notes, a decision-record entry added with --by operator, and a Discussion answer the support log's Interview shows the operator gave. State that a --by arbiter or --by parent entry and an auto-pick or Auto-mode assumption never count. Mirror that definition in the brief's new settle condition and in D1's Bound.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D1 (escalation brief changes)
    summary: The brief keeps step 1 about recording each design call the fork makes, which conflicts with D1's rule that the fork only applies a recorded operator decision
    fix: Add a bullet to D1's list of escalation-brief changes. It rewords the brief's step 1 to record the operator decision the fork applied, citing where it is recorded, with `lyx loom decision add ... --by parent`, instead of a design call the fork makes itself. Without a decision record, the fork names the applied decision and its source in its report.
---

### [MEDIUM] D7's #673 drop names an actor that does not exist on a clean merge

**Location:** `_lyx/discussion/decision-record.md` D7 third bullet, Acceptance criteria last bullet, Open risks last bullet.

**Issue:**
D7 says that if the parent's `CLAUDE.md` no longer holds the #673 workaround at Merge-In, "the Merge-In resolution drops it".
Merge-In is a Go engine row (`contracts/recipes/loom-recipe.yaml`, `engine: MergeIn`).
It merges, verifies and writes the merge record, and no agent resolves anything unless the merge conflicts.
Consider the ordering the focus asks about: `reed-add-parentless-placement`, now `running · Plan-Bouncer` on the board, lands its "workaround is removed in the same change" before this run lands.
Both branches delete the same `## Workarounds` lines from `CLAUDE.md`, and `ORCHESTRATOR.md` is a file only this branch adds.
So the merge-in is most likely clean, nothing triggers a resolution, and the #673 entry lands in `ORCHESTRATOR.md` after its fix.
The parent lock's reroute to Merge-In (`PATTERN-parent-lock`) ensures that a merge-in happens before landing, but nothing in that merge-in inspects `CLAUDE.md`.
The Merge-Review rubric covers how the task's code interacts with what the parent gained, and nothing tells it to apply D7's condition.
So no Decision or action owns the case, and the matching Acceptance criterion has nobody who could meet it.
The other ordering, where this run lands first, has an owner only as a board note in Open risks.
The plan writer does not schedule from Open risks.

### [MEDIUM] D1's settle bound leans on "recorded operator decision" without separating it from agents' recorded calls

**Location:** `_lyx/discussion/decision-record.md` D1 first bullet, the brief's new settle condition, and D1's Bound.

**Issue:**
D1 tells the fork to answer the envelope's question "from the operator's recorded decisions: the board entry, the decision record and support log, and the operator's board notes".
It settles only "when a recorded operator decision answers the envelope's question directly".
The decision record is not operator-only.
`lyx loom decision add --help` gives `--by parent, operator or arbiter`, and the arbiter stencil (`loom-template-arbiter.md`, "What you may change") lets the arbiter record design calls with `--by arbiter` before it hands on.
In an autonomous run, every Discussion decision is the writer's auto-pick, as this run's own support log shows ("Autonomous mode: no operator answered").
As worded, a fork can take an arbiter's, a parent's or the writer's own call as the operator's and accept on it.
That is the call the arbiter handed on because the board entry, the decision record and the support log did not record an operator decision for it.
So the bound the attack-surface rule requires is stated but has a hole.
The focus's other bound questions hold: the commands are limited to `decide_options`, `decision add` and `resume`, the scope and design-wrong limits are stated, and the brief's settle condition mirrors them.

### [LOW] The brief's step 1 still invites the fork's own design calls

**Location:** `_lyx/discussion/decision-record.md` D1, "The escalation brief changes to match".

**Issue:**
D1 changes the brief's materials list and settle condition only.
The brief's `## What to do` step 1 (`contracts/stencils/bouncer/bouncer-template-escalation.md`) stays: "Record each design call you make with `lyx loom decision add …`".
Under D1 the fork makes no design call of its own.
It applies a recorded operator decision or reports back.
The unchanged step therefore contradicts the new settle condition in the same file the fork reads.

## Rejected

- Focus 1, overlap between D1 and D2: not a defect.
  D1's signal is a `hand-on` with an empty `cause` in the attempt's `envelope.json`.
  `internal/loomcli/arbiter.go` `handOn` always sets a non-empty `cause`, and `internal/loomengine/arbiter.go` documents an empty `cause` only for the agent's own verdict.
  D2's `budget` is the Bouncer escalation cause, not the envelope's.
  According to the eligibility rule in `internal/loomcli/doc.go`, the arbiter admits only `hard-cap` and `circling`, so a `budget` escalation is refused and leaves no envelope.
  The two sets are disjoint.
  The path exists in every D1 case, because `finish` writes `envelope.json` into the attempt directory on every concluded call.
  Both the LLM driver's stop report (`verdict_path`) and the Go driver's report (`verdict file`) name a path inside that directory.
- Focus 3, D3: not a defect.
  Each peer escalation is classified by D1 and D2 first, the joint fork is held to D1's settle rule and bound, and the late peer's fresh fork starts only after the first fork has reported and been re-checked, so it reads a finished outcome, not an open fork.
