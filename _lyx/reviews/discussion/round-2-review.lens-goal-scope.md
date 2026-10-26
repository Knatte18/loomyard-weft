```yaml
lens: goal-scope
outcome: reported
```

Lens: goal-scope. I checked the decision record against its board entry, `beam-start-and-ownership` (T4).

**Goal points of the entry, and whether the record meets them**

- `lyx beam start [slug]` starts the dobby, reconciles the branches, ensures the beam window and starts the orch with its watcher: met, D1. The record reconciles before the dobby starts. The entry lists jobs, not an order, so this is no departure.
- The orch resumes its recorded session, else starts from the last handoff note, else from the start stencil, and a session that cannot resume falls back to the note: met, D2.
- It starts only that beam, and `--all` starts every beam this machine owns: met, D1.
- It refuses in a task worktree, naming the beam and its path: met, D3.
- It is idempotent and starts only a missing part: met, D2, and the first acceptance criterion.
- `lyx s` runs `lyx beam start` then `lyx reed attach`, and a refusal attaches nothing: met, D4.
- `lyx beam stop [slug]` stops both parts and `lyx beam status` shows both: met, D4.
- A fresh orch comes from `lyx orch refresh`: met, D2.
- `lyx orch` keeps `stop`, `status`, `distill`, `refresh`, `withdraw` and `rules`: met, D4, which also keeps the hidden `watch` and `resume-context`.
- Removed are `lyx orch start` with `--continue`, `--handoff` and `--adopt`, `lyx orch cleanup`, `lyx prime` and `lyx beam promote`: met, D4 and D13.
- Ownership sits on projects and the root project: met, D5. D5 narrows it to beam projects and root, and states why: a grouping project has no worktree or dobby.
- A task has no owner of its own and follows its project: met, D5.
- `lyx adopt <project>` and `lyx adopt root`, and never a single task: met, D6.
- "There is no adopt of a beam": D6 states how it reads this. Adopt names the beam's project, and the pair, dobby and orch move with it.
- `lyx beam start` refuses a beam another machine owns, naming `lyx adopt`: met, D3.
- `lyx adopt` of a project starts its beam, dobby and orch: met, D6. D6 skips this under the landing hold and states why.
- An adopt moves that project alone, and a nested beam keeps its owner: met, D6.
- The parent's dobby runs a beam's landing, and a machine lands only while it also owns the parent: met, D10, D11 and D6's resume rule. The support log's Turn 6 walks the G/H example.
- A beam's life is a batten run over its project entry with `recipe: pawl`: met, the Goal, D5, D8 and D10.
- The pawl rows Beam-Up, Beam-Await, Parent-Align, the Code-Review perch, Beam-Describe, Finalize and Beam-Down: met, D8 and D9. Finalize resolves conflicts in the conflict session and never reroutes.
- `lyx beam create <slug>` runs from the parent worktree, marks the entry a beam with its owner, starts the run at once and no longer makes the worktree: met, D10.
- `lyx beam land` ends Beam-Await, refuses with open members or member pairs, and refuses on a machine that does not own the parent: met, D10.
  D10 also refuses a beam this machine does not own. That follows from the land request living in the beam worktree's `.lyx/`, so it is no unlicensed work.
- Today's separate landing run of `lyx beam land` goes: met, D10.
- Batten's start guard lets a beam project start: met, D11.
- The "beam can land" notice after the last member lands, naming `lyx adopt` of the parent when another machine owns it: met, D12.
- The entry's brief says the overview window becomes `beam: <slug>`: already landed in afd030a4, and the record lists it under Out.

**Decision notes**

The board list has no note titled `Decision for beam-start-and-ownership:`, so there were none to read, and none the list names that I could not reach.
The record has no `Added after Discussion` entries.

**Findings**

None.
I found no goal point left out, no work belonging to another entry, no direction the entry rules out, no unlicensed work and no deferral.
D7 (the watcher stops the orch), D11's cancel refusal, D12's notice episode record, D13's removal of `ResetRecordsToParent` and the legacy migration in D10 all follow from goal points above.
The Out items cite their own notes: `loom-finalize-merges-itself`, `adopt-check-visible`, `readme-per-project-files`, `beam-promote-keeps-discussion` and `loom-beam-darns`.

Tool calls:
1. `lyx board list`, filtered for this task's decision notes: my context held the entry but not the board list, which the lens requires reading once.
