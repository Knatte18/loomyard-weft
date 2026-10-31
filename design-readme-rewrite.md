# README rewritten as a teaching text, with the overview figure brought up to date

`readme-rewrite` · Task · enhancement, docs · running · Worktree-Create

## Goal

`README.md` is rewritten from scratch as a teaching text for a reader new to LoomYard; the current one does not teach well and is out of date.

## What it covers, in this order

- What LoomYard is and the problem it solves, in plain words.
- The core concepts in the order a reader meets them: hub, worktree, board, task, run, orch and dobby.
- How a task goes from the board to a landed commit.
- How to get started.
- Detail goes to `docs/` by link, never repeated in the README.

## What it leaves out

- No list of modules by their weaving names: the names tell a new reader nothing.
  A weaving term appears only where the reader needs it, explained where it first appears.

## Up to date

Every claim is checked against the code and `lym help`: what is wrong is corrected, what no longer exists is dropped.

## The overview figure

`docs/loomyard-overview.svg`, linked from `docs/overview.md`, is out of date as well: for one, it has the orch start the runs, which dobby does now, and it shows landing rows that changed.
It is brought in line with the code, and the README shows it where it helps the reader.

## The old README

The current `README.md` is kept beside the new one as a backup file of its own, `README.old.md`, unchanged, so the operator can compare the two.
