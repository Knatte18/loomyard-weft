# Re-measure the batching hook and the edit-round rule on fresh runs

Adds `docs/research/turns/batching-hook-remeasure-2026-11-05`, a report with the scripts and tables behind it, and a line for it in `docs/research/README.md`.
It re-runs the turn-savings measurement on the runs that landed after `batching-hook-reach`, reading finished transcripts only; no agent run was made.

The report finds the hook still shows little effect and recommends removing it.
It fired 13 times in 22 runs (9 in main sessions, 4 in forks), and the next message held 2 or more calls after 5 of them.
That is about 2.5% of the lookup and edit breach turns it targets, and no breach share fell where it now reaches: edit follows rose from 4.2% to 6.9% of context against the previous sample, and in Webster forks from 6.9% to 11.2%.
A replay of the hook on one recorded edit follow fires, so the live gap between 156 edit follows and 2 edit firings is unexplained; the report names it as the one open question and does not test the candidates.

The edit chains split by what preceded them: none of the 71 chains with an edit follow comes after a failed gate or validator, and all 156 follows are mid-round, 105 of them in forks.
The sample holds 17 failed gate and validator results, so the rule "make all of them in one message" holds where a gate fails and fails mid-round in every role that edits.

## Design decisions

- The folder is new and dated; the 2026-11-04 study is untouched.
  Its sample becomes this study's baseline (`baseline-slugs.txt`, `baseline-recipes.tsv`), so the comparison is before and after the hook reached forks, and its figures reproduce.
- The sample is the 20 most recent darn runs and the 2 tapestry runs that landed after `batching-hook-reach`, about a day of landings, not the week the entry names.
  Only 2 tapestry runs landed, so the tapestry and fork figures rest on `raddle-recipe` and `pawl-orch-landing`; the report says so in its limits.
  The entry's own cap bounds the sample ("at most 20 darn runs and 5 tapestry runs"), so the run went ahead rather than stopping on the shorter window.
- `edit-chains.sh` and `edit-chains-summary.sh` are new: they group edit-follow turns into chains and name what preceded each by the last gate, validator or lint result.
  `events.jq` gains a `check` event for validator and lint results, since the rule covers "the round's gate, or its validator when it has no gate"; the `gate` rows are unchanged, so `gate-loops.sh` reads the same data.
- The recommendation to remove the hook follows the entry's rule for a hook that still shows little effect.
  The report states that removal is a separate change and files no note.
