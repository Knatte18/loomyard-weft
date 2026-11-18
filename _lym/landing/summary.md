# Report what the per-test coverage maps are good for

Adds a research report under `docs/research/speed/testmap-use-cases-2026-11-18/`.
It comes with two data tables and an entry in `docs/research/README.md`.
No code changes.

The report answers board task `testmap-use-cases`.
The gate no longer selects tests from the coverage maps of `internal/testcover`,
and `lym gate redundant` proved a reading filter, so the task asks what the maps are for.

## What the report holds

- **What the stored maps hold.**
  Counts read with `jq` from the hub's shared maps, 212 files, most at `5dc7a587`:
  tests per function, functions per test, functions no test reaches, functions only another package's tests reach.
  The per-package and per-test figures are in `data/per-package.tsv` and `data/tests.tsv`.
- **How fast a map goes stale.**
  The one test run the task allowed is `lym gate test --wait --map ./internal/boardengine` at `dcc5e7ac`.
  Its map is set against the stored map of the same package from `5dc7a587`.
  Four commits and 1,410 added production lines lie between the two.
  The stored map holds 17,376 of the fresh map's 17,640 pairs of a function and a reaching test, 98.5 percent,
  and 17,376 of its own 17,511 pairs are still true, 99.2 percent.
  263 of its 264 misses belong to a test it lacks or to a function in a file whose recorded hash differs.
- **Every use found**, 17 of them, in one table.
  Each has its source, what it needs from lym, a build cost in darns and the expected gain.
- **A ranking**, with the top three argued in full and against doing without the maps:
  1. a mutation check of the code a diff changes, as a gate report step for the code review;
  2. a reach note for a diff: the tests that reach each changed function, and the changed functions no test reaches;
  3. failure triage, telling a fixer whether a failing test reaches anything the run changed.
- **A plain statement for each use that is not worth it.**
  These are test selection and ordering in the gate, suspiciousness ranking,
  cutting tests by coverage beyond today's reading filter, learned test selection, a conflict forecast between runs,
  code ownership and generated "tested by" documents.

## Design decisions to check

- The ranking is by expected gain, and the report gives a different build order.
  The reader of use 2 comes first, since uses 1 and 3 read through it.
- The report ranks failure triage third and says why: the gate fixer ran in 92 of 291 darns at $0.27 a run,
  and an on-demand coverage run of the one failing test would serve nearly as well as the stored map.
- The triage evidence is four flaky landing failures, checked at file grain against maps built on 2026-11-15.
- The mutation check's cost per diff is an estimate.
  It rests on the redundancy survey's 0.49 s a mutant and the size of landed diffs.
  No such step was run, and the report names a trial over landed diffs as its first step.
- Figures quoted from papers come from their abstracts or summaries of them, each linked in the table of uses.
- The measurements were made with `jq`, `git` and shell over existing files.
  The report's Method section describes them, and no script is committed.

## To check or run by hand

- Read the top three against the hub's plans for `redundancy-sweep-existing` and the `code-cleanup` project.
  The report touches both and changes neither.
- The fresh `boardengine` map stays in this worktree's own map directory; nothing under Bolt was written.
