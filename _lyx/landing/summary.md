# Small fixes: tokencount, loom seeding help, plan rubric, batten notices

Five independent fixes.

## tokencount

- A working interval no longer counts when it runs into a turn-start line (a user line holding text and no tool result, other than an interrupt marker, a `<local-command-stdout>` line or a compaction summary) or is longer than a 30-minute stall cap.
  Such an interval is idle whole, so a crash followed by `--resume`, or a suspended host, stops inflating working time.
  The cap is one named constant, three times the harness's ten-minute cap on a foreground Bash call.
- A run counts only the transcripts that started after the slug's previous run.
  The lower bound is the commit date of the newest archive tag for an in-flight run, and of the next-older tag for a finished one; a slug with no such tag has no bound.
  A transcript judged by its earliest user or assistant timestamp, or one with no such line, is left out whole with its sub-agents, from tokens, cost, time, sessions, forks, Merriam starts, the begin-batch base and the orch window.
  The run's section says how many were left out and the bound's date.
  There is no upper bound and no flag to turn the filter off.
- The package doc in `main.go` describes both rules.

## loom

- `lyx loom step --help` no longer claims to bootstrap exactly as `lyx loom start` does.
  It says step refuses without a seed, and names both routes: `lyx loom start`, and the driverless `lyx shed seed self --recipe loom --driver go --param parent=<parent-branch>` followed by `lyx loom step`.
  `lyx loom run --help` names the same two routes.
- The missing-seed refusal for `run` and `step` names both routes; `status`, `pause` and `goto` still name `lyx loom start` only.
- `run`'s status-file refusal names `lyx loom step` beside `lyx loom start`.
- The bootstrap's disagreeing-seed refusal no longer says to keep the existing seed and drive it, which looped.
  It names `rm <seed path>` and a re-seed with the recorded parent.
- `contracts/specs/refusal-spec.md` has the changed `seed missing` row and new rows for the last two refusals.
- No behavior of `step` or `run` changes: there is no auto-seeding and no new flag.

## Plan review rubric

The "Proposed fixes" section of `loom-rubric-plan-review.md` names `redundant-package-target` and `redundant-file-target` as the checks a fix that adds or pairs targets must pass, with two claims pinning it in `claims_test.go`.

Closes #542

## batten

The quiet, api-error and driver-dead notices lead with the child's state and row again, for example `running at Plan-Write; agents idle for 25m0s`, and drop `while the child is running`.
Episode keys hold the condition name, so a batten restart does not resend a notice sent under the old wording.
`internal/battenshed/doc.go` and the `docs/overview.md` sentence on batten's `step` follow.

## Design decisions

- Whole-interval idling over clipping: an interval over the cap, or into a turn-start line, drops whole.
  A foreground call that blocks longer than 30 minutes loses that interval from the waiting session, while a sub-agent's own transcript still counts its work.
  A stall under the cap that ends at an assistant, `tool_result` or interrupt-marker line still counts as working.
- Compaction counts as working time, since lyx triggers it through `lyx orch distill` and refresh cycles.
- The bound judges a transcript by its first line, so an older run's session stays out even when its last lines postdate that run's final tagged commit.
  A session the previous run started after its final tagged commit would still be counted.
- Reports regenerated for reused slugs change figures; earlier reports are not comparable for those runs.
- The driverless seed uses the placeholder `<parent-branch>`; a wrong value writes a seed the next `lyx loom step` refuses at its seed stage, and the new `rm` and re-seed remedy clears it without recording a different parent.

## To check by hand

- The `tmux` subtest added to `TestSmokeStepBootstrapWiring` compiled but was not run here; run it in the tmux tier.
- The tokencount package-doc and launcher-description edit for the new time and transcript rules: `tokencount.sh` and `tokencount.cmd` were not changed, so confirm neither describes how time or transcripts are measured.
- The plan-level `lyx gate test ./...` and `lyx gate test --tags integration ./...` were not run as part of this description.
