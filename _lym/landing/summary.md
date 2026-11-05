# Bring README.md up to date with what lym does today

The root `README.md` described an earlier LoomYard.
It is rewritten to describe the system as it runs now, with the setup steps moved out into their own page.

## What changes

- `README.md` is rewritten.
  It opens with what LoomYard is and the design choices behind `lym`, follows one task from the board to a landed commit, and lists what goes beyond the run: gates, review lenses, parallel tasks and beams, `creel` messaging and the long-lived orch.
  An "Under the hood" part covers the hub, the board, beams and a table of names.
  Codex support is stated as under development, and a Slack integration for `creel` as planned.
- The research section, headed "Measurements from its own runs", has a table of four studies from `docs/research/` with their sample, finding, recommendation and status.
  A short "Open data" part points to the public records repository, `loomyard-weft`.
- `docs/getting-started.md` is new and holds the build, hub and first-task steps.
  The README links it.
- `docs/loomyard-run.svg` is new: a simpler figure of the run that uses only terms the README teaches, and the README embeds it.
- `docs/loomyard-overview.svg` is redrawn for the current pipeline and stays the figure of `docs/overview.md`.
- `README.old.md` is deleted; `README.md` supersedes it.

Only documentation changes; no Go code is touched.

## Design decisions

- The README is for understanding what LoomYard is, so setup lives in `docs/getting-started.md`, and the large overview figure stays in `docs/overview.md`.
- Each thing has one name.
  The second git repository is the "records repository" throughout; "piggyback" appears once as a description.
- The README gives no tokens, costs or time per run for LoomYard's own usage, and states the number of landed runs in one plain sentence.
  Study findings in the table keep their numbers.
- The README makes no claim about what Claude knows or does well; the `--help` line says only that the commands form a Cobra tree.
- The `docs/research/` tidy-up is not part of this change.
  The README links `docs/research/` and the four reports as they are today.

## To check by hand

None of this was run.

- View `README.md` on GitHub and confirm that `docs/loomyard-run.svg` renders at page width and that the in-page anchors (`#names-at-a-glance`, `#the-board`, `#measurements-from-its-own-runs`) resolve.
- Open the four report links in the table, the `loomyard-weft` links and the `archive/driver-arbiter` tree link, and confirm they resolve.
- Run the commands in `docs/getting-started.md` (`./update-plugins.sh`, `lym fabric clone`, `lym s`, the two `lym board` calls) against a scratch hub and confirm they behave as written.
- Confirm that the README's claims match the code: the default review budgets (2 for the decision record, 3 for the plan, 1 for the code) and the four recipe names.
- Confirm that `docs/loomyard-overview.svg` matches the recipes and processes it shows.
- Confirm that the "more than 700 runs" figure still holds at landing.
