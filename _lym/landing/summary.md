# Add a module label for every uncovered package group under internal/

This change makes no edit to the repository.
The board's labels are hub config in the hub's `_board/_lym/config/board.yaml`, outside the task worktree, and the repository's `internal/boardengine/template.yaml` ships `labels: {}`.

## What changed

The parent added 22 labels to the hub `board.yaml` with `lym config board --set`, one per package group under `internal/` that had no covering label.
Each description names its packages: `tender`, `hibernate`, `drift`, `darn`, `arbiter`, `landing`, `cleanup`, `termwindow`, `logger`, `git`, `cwd`, `friction`, `lenses`, `reconcile`, `preflight`, `parsers`, `process`, `fsutil`, `cli`, `pawl`, `lint` and `profiles`.
Packages already under an existing label got no new one.
`lym board labels` lists the result.

## Design decisions

- The labels are applied by the parent from a list this run sent, because the run may not edit outside its worktree.
- Small support packages are grouped by concern into `cwd`, `fsutil`, `cli` and `process`, instead of one label per package, so an entry can choose among the labels.
  The grouping is a judgment call and can be split or renamed.
- A package added later has no label until one is added.
