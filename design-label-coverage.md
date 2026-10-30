# A module label for every package group under internal/

`label-coverage` · Task · enhancement, board · running · Drift-Verify

## Seen (operator and hub orch, 2026-10-28)

Some lym modules have no module label, so an entry that touches them takes the nearest label, which says something else.
Note `landing-watch` took `workspaces` for dobby until the `dobby` label was added on 2026-10-28.
`lym board labels` lists the module labels; `ls internal/` lists the modules, and a package with no label covering it today includes, among others, `tender` and `hibernate`, `drift`, `darn`, `shedarbiter` and `arbiterstop`, `landingshed`, `cleanup`, `termwindow` and `logger`.

## Design (operator, 2026-10-29)

Every module gets its own label.
Go through the packages under `internal/` once and add a label for each module that has none, with a description naming its packages, so the label an entry needs exists when the entry is written.
