# Fixer report — discussion round 1

Every finding in `round-1-review.md` is fixed in `_lyx/discussion/decision-record.md`, with two support-log interview entries aligned to the revised decisions.

## Changes

- **F1** — D6's AskUserQuestion sentence now states the steer itself: state the question as the final message and end the turn without writing the result file.
  D6's rationale says why the steer is stated in full; the acceptance criterion pins it.
- **F2** — D7 now requires a single-line notice free of line breaks, single quotes, double quotes and backslashes, with a rationale grounded in the notice's path (`shell.Shell` quoting, `tmux send-keys` submitting at a line break, pwsh native-argument passing).
  The acceptance criterion covers line breaks in the assembled notice.
- **F3** — D6 now requires every sentence to hold inside forks, and the fork-run sentence tells a fork not to spawn further subagents.
  The acceptance criterion pins the no-re-delegation clause.
- **F4** — the webster-guard exclusion keeps its decision, with the rationale replaced: webster's fork stencils already carry the rule.
- **F5** — D6's resume-line rationale now cites the compaction re-render and the recording-not-enabled case;
  Open risks bullet 1 is removed and its fact folded into that rationale.
- **F6** — D4 is pinned to an append-only amendment; appending nothing leaves the prompt byte-identical.
- **F7** — the `plan.go` file comment fix moved into Scope's docs bullet and the docs acceptance criterion; the notes bullet is removed.
- **F8** — the duplicate Auto-mode assumption bullet is removed; Open risks keeps the restarted-run risk.
- **F9** — D2 now says `00-overview.md` is never copied from the archive and is written fresh, last, with `approved: false`, with the reason in its rationale;
  Open risks bullet 2 is removed.
- **F10** — D6's enumerated agent list is replaced with "every session `claudeengine` spawns".
- `support-log.md` interview entries 8 and 9 now match the revised D6 and D7 reasoning.

## Deferred

None.
