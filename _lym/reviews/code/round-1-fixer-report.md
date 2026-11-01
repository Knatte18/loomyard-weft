# Fixer-report — code review round 1

## Changes

All changes are in `contracts/stencils/burler/burler-fork-reading.md`, wave 1 and wave 2 (lines 22-23).

- **F1** (callees, other-process paths, failure and resume paths had no named call): fixed differently.
  - Proposed change not made: add a callees search and a failure-path search to wave 1.
  - Why: the self-check's fix-proposal lens showed that callees and failure, restart and resume paths have no search key until the definitions are read, so under the stencil's own rule "a call goes to a later message only when its arguments come from an earlier result" they belong in wave 2.
  - Made instead: wave 1 gains a per-changed-symbol search for the paths that reach it from another process; wave 2 now reads the callees of each changed symbol and the failure, restart and resume paths of each change that the definitions show.
  - Commits: the as-proposed edit, then the corrected one.
- **F2** (wave 1 dropped the at-definitions limit): fixed differently.
  - Proposed change not made: add "at the definitions of the symbols it cites" to the wave-1 file read.
  - Why: a bounded read needs offsets that only a search for the definitions yields, so in wave 1 the phrase could only be met by a whole-file read.
  - Made instead: wave 1 issues one search per file the target names for the definitions of the cited symbols, and wave 2 reads those definitions.
- **Self-check addition** (fix-affected-set lens): wave 1's per-symbol search now reads "one search per symbol or output", so the checklist's output readers and assertions get a first-wave call.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal, LOW, F1 callees search has no arguments in wave 1: fixed with my own change (callees moved to wave 2).
- fix-proposal, LOW, F1 failure-path search has no search key: fixed with my own change (moved to wave 2).
- fix-proposal, LOW, F2 bounded read cannot be issued in wave 1: fixed with my own change (wave 1 searches for definitions, wave 2 reads them).
- fix-affected-set, LOW, output bullet had no wave-1 call: fixed as proposed.

No fork returned without a report.

## Disputed

None.

## Deferred

None.
