# Card 4 — orchestrator-md

**Create:**
- `ORCHESTRATOR.md`

**Edit:**
- `CLAUDE.md`

**Intent:** Move this repository's orch-only rules out of `CLAUDE.md` into a new tracked `ORCHESTRATOR.md` at the repository root (D7), point at it from `CLAUDE.md` (D8), and leave the #673 removal an owner in either landing order (D9).
Every line is one sentence; table cells and blockquotes stay on one line.

`ORCHESTRATOR.md` opens with `# ORCHESTRATOR.md — orch-only rules for loomyard` and one sentence: these rules add to the orch's role for this repository, and lyx renders the file into the orch's role file at every delivery.
Its sections, worded as rules for the orch:
- `## Production`: an exception to `CLAUDE.md`'s Production rule: after your own pushed fix, on a clean `main`, you run `update-plugins.sh` yourself and say so in your report.
- `## Watching runs`: the operator watches each run in its own terminal window, never in VS Code; VS Code opens only when the operator asks for it, through `lyx ide` or `batten.yaml`'s `open_ide`.
- `## Workarounds`: the #673 entry, moved verbatim from `CLAUDE.md`: `lyx reed add` with no `--parent` uses its default anchor `below-parent` and splits the new strand into the selvage `shell` window; until #673 is fixed, pass `--anchor own-window` to every `lyx reed add` that names no `--parent`.

`CLAUDE.md`:
- `## Production`: the sentence "The one standing exception is the hub orchestrator in the prime: …" goes, and "The second standing route is the alignment agent's approve at PR-Gate: …" is reworded so it leans on nothing moved: batten's Land row runs the hub's `deploy` command after every landing, including one the alignment agent's approve at PR-Gate makes with no orch turn; `alignment_pr_approve: false` in the hub's `loom.yaml` withdraws that approve's landing.
  Every other sentence of the section stays.
- `## Watching runs` goes whole: its orch-directed sentences move to `ORCHESTRATOR.md`, and its description of batten's `Open-Window` row is cut, since `internal/battenshed/doc.go` and the role's run loop carry it.
- `## Workarounds` goes whole, its #673 entry moved to `ORCHESTRATOR.md`.
- `## Notes go in git, not file-memory` gains one line: a rule or workaround only the orch needs goes in `ORCHESTRATOR.md`, never in `CLAUDE.md`.
- Every other section, `## Terminology` included, stays.

D9, this run landing first: the implementer of this card runs `lyx board upsert` from the task worktree with a payload carrying the fields below, since `reed-add-parentless-placement`'s own entry is run-held and never edited.
The write touches only the new note's own entry, never this task's entry nor `reed-add-parentless-placement`'s; a re-run upserts the same slug, so it cannot duplicate the note.
The note is kind `note`, slug `reed-673-workaround-home`, labels `bug` and `reed`, titled "Where the #673 workaround removal lands", and names `reed-add-parentless-placement` in its body.
The body, worded for either landing order: that task's workaround removal targets `ORCHESTRATOR.md`'s #673 entry wherever the parent holds it there, and `CLAUDE.md`'s otherwise; on that task's `landed` notice, when the parent's `ORCHESTRATOR.md` still holds the #673 entry, the orch hands a fresh agent its removal.
D9's other order, `reed-add-parentless-placement` landing first, needs no card: this run's Merge-Review perch measures each merge-in against the decision record and raises a #673 entry left in `ORCHESTRATOR.md` once the parent's `CLAUDE.md` no longer holds the workaround.

**ImpactSummary:** Docs only: one new root file the orch render reads by path, three `CLAUDE.md` sections cut or reworded and one line added, and one new board note; only `internal/lyxcwd`'s vocabulary scan reads `CLAUDE.md`.

**Verify:**
```bash
lyx board get '{"slug":"reed-673-workaround-home"}' --body
```
