```yaml
lens: goal-scope
outcome: reported
```

Task: goal-scope lens. I checked the decision record against the board entry `beam-start-and-ownership` and looked for `Decision for` notes.

**Goal points of the entry, and whether the record meets them**

- `lyx beam start [slug]` starts the dobby, reconciles, ensures the window and starts the orch with its watcher: met (D1).
- The orch resumes its recorded session, else starts from the note, else from the stencil, and a session that cannot resume falls back to the note: met (D2).
- Only that beam starts, and `--all` starts every owned beam: met (D1).
- It refuses a non-beam folder, naming the beam and its path, and refuses a beam another machine owns, naming adopt: met (D3).
- A live part is left alone and only a missing part starts: met (D2, acceptance criteria).
- `lyx s` runs start then attach, and a refusal attaches nothing and leaves its error: met (D4).
- `lyx beam stop` and `lyx beam status` cover both parts: met (D4).
- A fresh orch comes from `lyx orch refresh`: met (D2).
- `lyx orch` keeps `stop`, `status`, `distill`, `refresh`, `withdraw` and `rules`: met (D4).
- Removals (`orch start` and its flags, `orch cleanup`, `prime`, `beam promote`): met (D4, D13).
- The root project is driven by the owner's prime: met (D5).
- "Each project is owned by one machine": narrowed to beam projects and root. D5 states the narrowing with its reason, and the auto-mode assumptions record it. I accept this as settled by the writer.
- A task has no owner: met (D5).
- `lyx adopt <project>` and `lyx adopt root`, never a single task: met (D6).
- "There is no adopt of a beam": not addressed; see finding 1.
- `lyx beam start` refuses a beam owned elsewhere: met (D3).
- An adopt starts the beam's dobby and orch: met (D6).
- An adopt moves the project alone, and a nested beam keeps its owner: met (D6).
- Landing is the parent beam's job, and a machine lands only while it owns the parent: met (D10).
- The pawl recipe and its rows: met (D8).
- `lyx beam create` runs from the parent, marks the entry and owner, starts the run, and no longer makes the worktree: met (D10).
- `lyx beam land` ends Beam-Await and refuses on open members, member worktrees and a parent owned elsewhere: met (D10).
- The separate landing run goes: met (D10).
- Pawl is the heaviest recipe, with align and review: met (D8).
- Batten's start guard lets a beam project start: met (D11).
- The "beam can land" notice, naming adopt of the parent when another machine owns it: met (D12).
- The window rename: correctly marked Out, since it landed in `afd030a4`.

**Decision notes:** the board list holds no note titled `Decision for beam-start-and-ownership:`, so none was read and none was out of reach.

**Findings**

1. Severity LOW, class `consistency`, location D6 (`decision-record.md:89-100`).
   - Summary: the entry's line "There is no adopt of a beam." is neither met nor interpreted.
   - D6 makes beam project slugs the only project arguments `lyx adopt` takes. D3 and D10 name `lyx adopt <slug>` for a beam.
   - The record should say how it reads that line, presumably "no separate beam-level adopt; a beam moves only by adopting its project". Otherwise the plan writer meets a direct contradiction between the entry and D6.
   - Evidence: the board entry body, Ownership bullets (from `lyx board get '{"slug":"beam-start-and-ownership"}' --body`), and `decision-record.md:91,93`.

**Tool calls**

- `lyx board list`, filtered: for the entry list and any `Decision for` notes, which my context did not hold.
