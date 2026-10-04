# Card 10 — board-model-docs

**Create:**
- `internal/boardengine/doc.go`

**Edit:**
- `internal/boardengine/board.go`

**Prosa:**
- `docs/overview.md`
- `plugins/ly/skills/board/SKILL.md`
- `tools/sandbox/SANDBOX-CORE-SUITE.md`

**Intent:**
Docs for the new board model, and the area-label list D13 hands to the hub.

- `internal/boardengine/doc.go` takes the package doc out of `board.go` (PATTERN-documentation-lifecycle: a built design lives in its package's `doc.go`), rewritten for the new model: two kinds, task and note, only a task claimable and only a task carrying `depends_on`;
  `labels` validated against `board.yaml`'s `types` and `labels`, a note with exactly one type label and a task with one or more;
  the on-load migration from tier and type;
  the README layout;
  the `issues` record and the intake procedure (list, import or fold, close), with the board as the one list and GitHub as the inbox `selfreport` files to.
  Drop the closing "planned, next-up and someday work" line.
- `board.go` keeps only its file-level comment.
- `docs/overview.md`: the board line in the module list says each entry is a task or a note with labels, and names `lyx board intake`; the "planned, next-up and someday work" wording under Related becomes tasks and notes.
- `plugins/ly/skills/board/SKILL.md`: replace Tiers and Types with Kinds and Labels, show `promote` turning a note into a task, `upsert` with `kind`, `labels` and `--body-file`, `list`/`find` with `--label`, and a short Intake section (list, import or fold with `into`, close with a reason);
  the "never name a dependency in a body" rule now holds for tasks only, and a relation between notes is a sentence in a body.
- `tools/sandbox/SANDBOX-CORE-SUITE.md`: its `lyx board upsert` example passes `"labels":["enhancement"]`, which a note now needs.

D13: derive every area label the migration yields on loomyard's live board by running `lyx board list-full` and applying card 4's brief-prefix rule to each entry (a scratch program, never committed), and put the result in this card's commit message body as a ready-to-paste `board.yaml` snippet:

```yaml
labels: [undecided, <every area label, sorted>]
```

under the line `board.yaml labels for the parent, to write before deploying:`, as a record on the branch.
The commit message alone does not reach the PR description: the Describe session reads the decision record, the task's board entry, the diff and the run records, never commit messages.
So the same snippet also goes into a closing paragraph of `doc.go`'s migration section, headed `loomyard's own area labels`, saying the hub writes the list into the parent's live `_lyx/config/board.yaml` before deploying and removes the paragraph afterwards; the diff then carries it into the PR description.
The paragraph is phrased as a pre-deploy hand step for the hub ("before deploying the landed binary, write this list into the parent's `_lyx/config/board.yaml`"), not as a description of the migration.
The Describe session carries "anything to check or run by hand that was not run" from the diff into the PR body, so only an action-shaped paragraph reliably reaches the description.
The paragraph also asks that the PR description quote the snippet verbatim, because the hub copies the list from the PR description, not from the diff, and a summary that drops the list leaves the hub nothing to paste.
No weft-carry mechanism is added.

**ImpactSummary:** Documentation only, no code change: moves the boardengine package doc into `doc.go`, and the one temporary `doc.go` paragraph is a hand-off the hub acts on and then removes.
