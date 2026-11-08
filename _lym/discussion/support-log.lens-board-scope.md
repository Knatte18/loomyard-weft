```yaml
lens: board-scope
outcome: reported
```

Board-scope lens: the arras record checked against its board entry, the touching entries, the peers and the decision notes.

**Findings**

1. **MEDIUM** (class `decision`), D2 and Auto-mode assumptions. The entry says "raddle fixes what it finds", but the record uses a cold `BurlerFix` session as the split perch's fixer.
   - Evidence: the entry's Idea says "The split perch reviews the proposal against the Sketch … raddle fixes what it finds; the arbiter settles stops". D2 instead picks "a cold `BurlerFix` session". The record reads "raddle" as "the raddle stage" and defaults the fixer's model to the raddle model.
   - That is a reinterpretation of the entry's goal point, not a choice between options the entry left open.
   - Question for the parent: does a cold Burler fixer running on the raddle model meet "raddle fixes what it finds", or must the fixer be the Raddle-Write agent re-entered with the findings?

2. **MEDIUM**, D5 and D6. The entry's point "the project lists their order" is not carried into the design.
   - Evidence: the entry says "each task carries its `depends_on` and its part of the decision record, and the project lists their order".
   - D6 only has the project's body list the parts. No decision sets the project entry's `order` field, the board's member-order list on a project. `fileRaddleEntry` writes no `order` today.
   - The record should say the converted project's `order` holds the parts in their filed `depends_on` order, with an acceptance criterion for it.

3. **LOW**, D5 and Scope. Refusing shape `beam` and multi-part `tasks` narrows raddle's shape choice inside arras, and the record's Scope says raddle's choice is unchanged.
   - Evidence: Scope Out says "Raddle's choice of each part's shape, recipe and profile is unchanged". The peer `motif-before-tapestry` says "raddle chooses each part's shape, tapestry included, and that choice stands".
   - D5 is the entry's own two-outcome rule, and part recipes stay free, so this is no conflict with the peer. Scope should say plainly that the proposal's top-level shape is limited in an arras run.

**Goal points of the entry**

- One recipe runs Motif and then raddle in a row: met (D1).
- The perch reviews the split against the Sketch, its record and post-Discussion decisions: met (D2, D3).
- Raddle fixes what it finds: met only as reinterpreted (finding 1).
- The arbiter settles stops and only the rest reaches the orch: met (D2).
- Operator decisions go in through `lym loom decision add --by operator`: met (D7).
- The operator only releases the filed tasks: met (D4, D8).
- The recipe is named `arras`, `cartoon` stays reserved and the glossary is updated: met (D9, Scope Out).
- `arras` is the default for a tapestry-sized note, with `motif` alone kept for a split made later: met (D9).
- The shipped rule in board-policy and the orch text is rewritten: met (D9, with an Open risk on `motif-before-tapestry`).
- One task ends as one drafted task with its record, and the arras entry is done and pruned: met (D5).
- A family goes into a project named after the slug, each task with its `depends_on` and its part of the record: met (D5, D6).
- The project lists the family's order: not met (finding 2).
- Each family task is an ordinary darn or tapestry run with its own perch and landing: met (Scope Out).
- The closed-project idea stays out: met (Scope Out).
- Whether releasing a project releases all its tasks: settled yes (D8).
- Whether the project keeps the whole record: settled yes (D8).

**Entries, peers and parent commits checked**

- `motif-before-tapestry` (running Code-Burler-Review, changes `plugins/ly/skills/board-policy/SKILL.md`): overlaps D9 on the same text. The record covers this under Open risks, so there is no unrecorded conflict. Its decision record holds only the entry's goal text.
- `darn-cost-measure`: no conflict. It prices darns for arras and owns no arras design.
- `discussed-task-split`: no conflict.
  - Its line "The trial recipe `arras` is removed" is history of an earlier trial.
  - Its `stitch` idea runs the parts as well, while arras stops at drafts.
- `slack-push-channels`: no conflict.
- `reed-assist-pane-width`: no conflict.
- `slack-machine-bridge`: the entry cites it as the case for `motif` alone; D9 keeps that, so no conflict.
- Parent commits: `lym loom peers` listed none.

**Decision notes**

- No `Decision for motif-raddle-recipe:` note is on the board.
- There are no `Added after Discussion` entries.
- No note was unreachable.

**Tool calls**

- Read `motif-before-tapestry`'s decision record in its worktree. My context held that run's board entry and changed paths, but not its record.
