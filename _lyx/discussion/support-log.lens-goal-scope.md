```yaml
lens: goal-scope
outcome: reported
```

Lens goal-scope: checking the record against board entry `beam-start-and-ownership`.

**Decision notes:** `lyx board list` (already in context) shows no `Decision for beam-start-and-ownership:` note, and the record has no `Added after Discussion` entries. I could reach every note the list names.

**Goal points of the entry, and whether the record meets them**

- `lyx beam start [slug]`: starts the dobby, reconciles, ensures the window, starts the orch and its watcher. Met (D1).
- The orch resumes its recorded session, else starts from the note, else from the stencil, and an unresumable session falls back to the note. Met (D2).
- Only that beam starts, and `--all` starts every beam this machine owns. Met (D1).
- Refuses in a non-beam folder, naming the beam and its path, and refuses on a beam another machine owns. Met (D3).
- Idempotent: only a missing part is started. Met (D2).
- `lyx s` runs start then attach, and a refusal attaches nothing. Met (D4).
- `lyx beam stop` and `lyx beam status` cover both parts. Met (D4).
- "A fresh orch comes from `lyx orch refresh`." Only implied: D4 keeps `refresh`, but no decision states this path. LOW, see F3.
- `lyx orch` keeps its verbs, and `orch start` (with all three flags), `orch cleanup`, `lyx prime` and `beam promote` are removed. Met (D4, D13).
- The root project is driven by its owner machine's prime. Met (D5).
- Each project is owned by one machine. Partly met: grouping projects carry no owner (D5, auto-pick). See F1.
- A task follows its project. Met (D5).
- `lyx adopt <project>` moves the project with its tasks and runs, `adopt root` replaces `--prime`, and a task is never adopted. Met (D6).
- No adopt of a beam. Met: D6 adopts projects only.
- `beam start` refuses another machine's project, naming `lyx adopt`. Met (D3).
- Adopt starts the beam's dobby and orch. Met (D6).
- An adopt moves only that project, and nested beams keep their owners. Met (D6).
- Landing is the parent's job, needs the parent's owner, and the Neve/OSL-1046 example holds. Met (D6, D10).
- A beam's life is a batten run with the pawl recipe, and the entry carries `recipe: pawl`. Met (D8, D10).
- The row table: Beam-Up, Beam-Await, Parent-Align, the perch, Beam-Describe, Finalize, Beam-Down. Met (D8).
- `beam create` runs from the parent, marks the entry with its owner, starts the run and no longer makes the worktree. Met (D10).
- `beam land` ends Beam-Await and refuses on open members or member worktrees, or a parent owned elsewhere. Met (D10).
- The separate landing run goes. Met (D10).
- Batten's start guard lets a beam project start. Met (D11).
- The dobby sends a "can land" notice, which names `lyx adopt` of the parent when another machine owns it. Met (D12).
- Brief: the window becomes `beam: <slug>`. Met: the record cites it as already landed in afd030a4.

**Findings**

- **F1, MEDIUM, design (D5 and Auto-mode assumptions).** The record narrows the entry's ownership rule without flagging the departure.
  - The entry says "Each project is owned by one machine and driven by its beam there."
  - The record gives grouping projects no owner; their tasks follow the nearest beam or root.
  - This is defensible, since a grouping project has no worktree and no dobby. But D5 states it as fact, and only the auto-pick list mentions it.
  - Fix: state in D5 that this narrows the entry's "each project", and why, so the plan and review don't read it as an omission.
- **F2, LOW, scope (D7).** D7 adds a new mechanism the entry never asks for: the orch's watcher stops the orch when another machine owns its beam.
  - The entry's "Both run on the machine that owns the beam" licenses the intent, not this watcher mechanism.
  - Keep it if the writer judges it necessary, but the rationale should tie it to that sentence and to the adopt flow D6 walks through.
- **F3, LOW, consistency (D4).** The entry's "A fresh orch comes from `lyx orch refresh`" has no counterpart in the record.
  - D2 always resumes a recorded session, so the only way to get a fresh orch is `refresh` on a live orch.
  - Fix: one clause in D2 or D4 saying so.
- **F4, LOW, consistency (Open risks, first bullet).** The question "keep the removal?" is already settled: the entry orders `lyx beam promote` removed.
  - Fix: rephrase the risk to the two notes' premise alone, `beam-promote-keeps-discussion` and `loom-beam-darns`. Rewording those notes is a board matter for the parent, not a doubt about this task.

There are no deferrals, no work belonging to another entry, and no decision that edits this task's board entry.

**Tool calls:** none. My context already held `lyx board list`, this task's entry and the touching notes.
