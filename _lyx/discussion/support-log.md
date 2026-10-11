# Support log — small-fixes-batch

## Interview

The session ran autonomously; every turn below is a self-answered question, grounded in the code read during exploration.

1. **Darn `board get` form.** The darn stencil emits `lyx board get {{.slug}}`; `board get` takes JSON. The landing describe stencil carries the same bare form. Picked: fix both to the JSON form already used by the loom discussion stencils (D1).
2. **Darn spec path.** The darn stencil names `{{.specs_dir}}/final-summary-spec.md`, but the specs registry deploys only `loom-plan-spec`, under `loom/`. The board proposes the repo path `contracts/specs/final-summary-spec.md`. That path exists only in loomyard's own repo, while darn targets any hub's repo, and `PATTERN-stencil-ownership` routes specs through the deployed dir. Picked: rename to `loom-final-summary-spec.md`, register it, point the stencil at `{{.specs_dir}}/loom/` (D2). The `loom-` prefix follows the registry test's guard against one-file family directories.
3. **overview.md recipe line.** The batten paragraph's opening line already says `loom` or `darn`; a later sentence says "`loom` or empty". `battenshed` confirms `lyx loom start` serves both. Picked: rewrite the stale sentence (D3).
4. **Batten window marker.** InnerRun clears the terminal-opened marker whenever it finds no child status file, and the scratch dir is slug-scoped, so a vanished status file reopens the window. Picked: reset the marker at batten's fresh-status seeding instead, through a `battenshed` export (D4).
5. **Reconcile stale default.** `configsync.ReconcileAll` already runs per-module `Migrate` hooks and reports `Migrated`. Reconcile has no record of earlier templates, so "equals old default" cannot be detected without new state. Picked: floor-raising `Migrate` hooks for the three per-worktree modules whose loaders raise a value to a floor (D5). Batten's `poll_interval_s` is a recipe parameter and shed's idle window is not a config key, so both stay out.
6. **Destructive verbs.** `fabric prune` and `fabric cleanup` already use `--apply` with a dry-run default. Picked: the same flag and shape for `fabric unwire`, `board retire-legacy`, `board prune`; callers found in the board skill, the board README Done text and the sandbox suites; no Go caller (D6).
7. **Index note audience.** `clihelp.withNote` appends a note on any line, and a group line prints under any audience where a child prints; board's `rerender` and `sync` are internal. Picked: a note-audience annotation, a note without one never prints, plus a test over the live tree (D7).
8. **Carry-over on disk.** `WriteCarryOver` splices, checks the produced lines and renames, with no lock; `lyx loom decision add` writes the same file. A post-rename read alone catches only a writer that clobbers the carry-over, not one the carry-over clobbers. Picked: a pre-rename compare plus the post-rename read, both erroring, with a new transient refusal-spec row (D8).

## Rejected alternatives

- **D2, repo path `contracts/specs/final-summary-spec.md`:** works only when the hub's repo is loomyard; contradicts `PATTERN-stencil-ownership` and leaves `specs_dir` unused in darn.
- **D2, inline the format in the darn stencil as the landing describe stencil does:** violates `PATTERN-producer-pointer-rule`.
- **D2, register under the name `final-summary-spec`:** deploys into a one-file `final/` family directory, which the specs registry test is written to prevent.
- **D2, register `loom-final-summary-spec` while keeping the source file name:** splits the spec's name between `lyx stencil list` and the repo; the rename touches only a handful of references.
- **D4, clear the marker in the Worktree-Create row:** couples another producer to InnerRun's marker and runs on a resume that re-steps that row.
- **D4, key the marker on the child's run identity:** the child's status is exactly what vanished, so it cannot anchor the once-per-run rule.
- **D5, update values equal to the old template default:** reconcile has no record of past templates, and an operator value equal to an old default is indistinguishable from an untouched one.
- **D5, a generic floor table in `configsync`:** would declare each floor a second time outside its engine.
- **D6, `--yes` or `--confirm`:** a second spelling for the same guard that `fabric prune` and `cleanup` already name `--apply`.
- **D6, refuse a bare call with an error:** a dry-run report tells the caller what the verb would do, which a refusal does not, and matches the existing idiom.
- **D7, derive a group note's audiences from its children:** cannot tell which children the note describes; the board group has operator and internal children.
- **D7, per-audience note keys (`lyx.index-note.<audience>`):** more annotation surface than the one audience the only note needs.
- **D8, a lock shared by `AppendDecision` and `WriteCarryOver`:** beyond the board's ask; detection turns the silent lost update into an error, and a re-run is idempotent.
- **D8, post-rename read only:** misses the case where the carry-over overwrites another writer's bytes.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): Fix the landing describe stencil's bare `board get` too? Yes — same defect, same file family (A2).
- Q2 (resolved, auto-pick): Repo path or deployed spec for darn? Deployed spec, renamed `loom-final-summary-spec` (D2, A1); departs from the board's literal text.
- Q3 (resolved, auto-pick): Where does the terminal-opened marker reset? At batten's fresh-status seeding (D4).
- Q4 (resolved, auto-pick): Old-default or floor-based reconcile? Floor-based, through `Migrate` hooks, for `shuttle`, `orch`, `loom` (D5, A3).
- Q5 (resolved, auto-pick): Which floors are in scope? Only those on config-module keys; batten's recipe parameter and shed's idle window are out.
- Q6 (resolved, auto-pick): Flag name and bare-call behavior? `--apply`, read-only dry run exiting 0 (D6, A4).
- Q7 (resolved, auto-pick): How is a note scoped? One note-audience annotation; no annotation, no note (D7).
- Q8 (resolved, auto-pick): What does the carry-over check compare? Whole-file bytes, before and after the rename (D8, A5).
- Q9 (resolved, auto-pick): Does D8 need a refusal-spec row? Yes, loom is bound by `PATTERN-refusal-way-forward`; a new transient row.
