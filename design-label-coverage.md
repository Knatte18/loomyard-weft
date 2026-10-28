# A module label for every package group under internal/

`label-coverage` · Note · enhancement, board

## Seen (operator and hub orch, 2026-10-28)

Some lym modules have no module label, so an entry that touches them takes the nearest label, which says something else.
Note `landing-watch` took `workspaces` for dobby until the `dobby` label was added on 2026-10-28.
`lym board labels` lists the module labels; `ls internal/` lists the modules, and a package with no label covering it today includes, among others, `tender` and `hibernate`, `drift`, `darn`, `shedarbiter` and `arbiterstop`, `landingshed` and `pawl`, `cleanup`, `termwindow` and `logger`.

## Idea

Go through the packages under `internal/` once, and add a module label for each group of packages an entry can touch on its own, with a description naming its packages, so the label an entry needs exists when the entry is written.
Note `labels-from-diff` derives or checks labels from what a task touches; a label set that covers every package is what such a check maps paths onto.

## Open

- How fine the labels go: one per package group a task can touch alone, not one per package.
- Whether a label names its packages in its description, which a path-to-label check could read.
