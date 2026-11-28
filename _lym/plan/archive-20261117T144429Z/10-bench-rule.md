# Card 10 — The bench rule on the Codex project and the writer bench

**Custom:**
- `board:codexengine`
- `board:codex-writer-bench`

**Intent:**
D11, written to the board with `lym board upsert`, each body read first with `lym board get '{"slug":"<slug>"}' --body` and written back whole with `--body-file`; no code or file of the worktree changes.

The project `codex-engine-trial` was merged into project `codexengine` (P1) on 2026-11-16, whose body says so, so its Rules live there:
`codexengine`'s `## Rules` gains one bullet: a Codex bench runs the real producer with only the model swapped through `run_settings`, and never hand-builds Codex's prompt;
its report opens with the run's own manifest record, the copy kept under the run-directory root's `manifests` directory, and, for the engine that did not run, the output of `lym shuttle manifest` for the same seat.

`codex-writer-bench` (T15), the draft that replaced `codex-writer-bench-osl` on 2026-11-16, already runs the real Darn-Write producer and never hand-builds the prompt.
It gains a section `## Bench rule` that states the `codexengine` project's bench rule whole, as above: the writer's model swapped through `run_settings`, and the report opening with the run's manifest record from the `manifests` copy and `lym shuttle manifest`'s output for the Claude writer seat.
The section says that `codex-writer-bench-osl`'s hand-built prompt and scratch replay no longer hold, and that its replay route is the redesigned one under `## The one run`: the real producer in a bench worktree at the landed commit's parent.
It leaves `## The one run` as it stands, its one Codex session included, since that is this bench's own design and not the osl draft's single-session bound.

Neither write touches this task's own entry, files a task or promotes a note.
Review checks both bodies state the rule (D11's acceptance criterion); no test pins the wording.

**Commit:** 10: State the Codex bench rule on the board
