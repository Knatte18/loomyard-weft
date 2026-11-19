# Research: mutation trial of lym gate mutate on 20 landed diffs

Adds `docs/research/speed/mutation-trial-2026-11-19/report.md`, an index line in `docs/research/README.md`, and the full envelopes of the three largest runs under `data/`.
No code changes.

## What was measured

`lym gate mutate --base <first parent>` was run once on each of 20 landed `main` diffs that change Go production code.
The diffs span many packages and range from 11 to 5 427 changed lines.
Each ran in a hub pair checked out detached at the landing, so each verb run refreshed its own maps.

| | Mutants run | Killed | Survived | Unjudged |
| --- | --- | --- | --- | --- |
| All 20 diffs | 430 | 374 | 44 | 12 |

The runs took 604 s of wall time in all, 30 s at the median and 80 s at most.
The verb's cap of 40 mutants stopped 6 runs and left 725 planned mutants unrun, 623 of them on one diff of 195 changed functions.
One package was not judged, from a build temp-directory failure rather than the diff.

## What the survivors say

All 44 survivors were read in the landing's tree.
26 are a check a reviewer would act on, 6 are missing checks of low value and 12 no test could reasonably kill.
11 of the 20 diffs have at least one of the 26, 3 have only the other kinds, and 6 have no survivor.
The report tabulates every survivor with its file, line and judgment.

## Decision

The report recommends building the `mutation` gate report step as a later darn, in the template's `diff` keys, after three changes to the verb's bounds:

- spread the mutant cap across the changed functions instead of taking them in turn;
- skip log-only statements and branches of another platform as mutation sites;
- have the step print one summary line and keep the full list in `mutation.json`.

## Design decisions

- The report sits under `docs/research/speed/` beside the other coverage-map studies, which the research README groups by topic; the parent's path named no topic folder.
- The judgments are one reader's; no reviewer rated the survivors, and the report says so.
- The replays used a hub pair the parent checked out for each landing, because lym refuses a plain `git worktree` outside the hub.
- One run's uncovered-block list was cut off when read, and the run could not be repeated under the no-repeat rule, so its count is a lower bound.
