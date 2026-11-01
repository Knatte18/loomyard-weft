# A reviewer, fixer or writer waits for all its forks in one call

A Discussion reviewer, Discussion fixer or Plan fixer that forked its lenses ended its turn to wait, and woke once per fork result, since each fork's result arrives as its own message.
Each wake-up is a turn with no tool call at the session's full context: 79 such turns over three tapestry runs, 8% to 9% of the three roles' weight and 13% to 19% of their time, per `docs/research/review-turn-audit-2026-11-01/report.md`.

The forks keep running in parallel.
What changes is how the parent waits: it now waits inside one foreground tool call, `lym shuttle await-forks`, instead of ending its turn.
Claude Code queues a fork's completion notification that arrives during a tool call and delivers every queued notification in the turn after the call returns, so the parent takes all fork results in one turn.

## What changed

- **`lym shuttle await-forks [--cap]`** (`internal/shuttlecli/awaitforks.go`), a read-only role verb.
  It resolves the calling session's run from the strand name reed puts in the pane's environment, then waits until no fork the session launched is still running.
  It answers `ready: true` when none is running, and when `--cap` elapses first it answers `ready: false` with the forks still running.
  `--cap` defaults to `shuttleengine.DefaultAwaitCap`, under the 5-minute cache lifetime, like `lym shuttle await`.
  A caller outside a shuttle strand is refused with a way forward.
- **`Runner.AwaitForks`** and the poll `awaitForks` (`internal/shuttleengine/await.go`) read the outstanding forks at most once a second until none remain or the cap elapses.
  They read through a new optional engine capability, `OutstandingForkReader` (`internal/shuttleengine/engine.go`), so the Claude transcript format stays inside `claudeengine`.
  An engine without the capability gets an error, never an empty answer, since an empty answer would end the wait.
- **Claude's `OutstandingForks`** (`internal/shuttleengine/claudeengine/background.go`) reads the session's own transcript.
  It uses the existing unmatched-launch reading, narrowed to forks: a fork launch whose task id no later `<task-notification>`, queue-operation line or queued_command attachment names is still running.
  Claude Code writes the queue-operation line as soon as a notification is queued, so the reading turns empty while the parent is still inside the waiting call.
  A missing or unreadable transcript is an error.
- **Fork launches are background launches.**
  `isBackgroundLaunch` now counts an Agent or Task call whose input names `subagent_type: "fork"`, since Claude Code runs every fork in the background and its input has no `run_in_background` flag (the `master-fork-running` corpus transcript has this shape).
  This also corrects the Stop-payload transcript fallback, which used to miss forks.
- **Stencils.**
  `burler-cluster-fan.md` (the reviewer's fanned round), `burler-fix-self-check.md` (the fixer's self-check) and `twill-directive.md` (the writer's lens review) replace "ending your turn while forks are outstanding is allowed".
  Each now says to run `lym shuttle await-forks` in the foreground with the Bash tool's `timeout` at 300000, never end the turn while a fork is outstanding, and run it again in a turn that does nothing else while it answers `ready: false`.
  The fixer and the writer run it in the message after the spawn.
  The reviewer runs it after its holistic review, before consolidating.
  When the verb fails, the session ends its turn instead and takes each result as it arrives, the old behaviour.
- `lym shuttle state` and `await-forks` share one renderer, `taskFields`, for a list of background tasks.
- Docs: the `shuttleengine`, `claudeengine`, `burlerengine` and `twilldirective` package docs and the shuttle row of `docs/overview.md`.

## Design decisions

- **A blocking wait over the transcript, not fork-written marker files.**
  Forks could write their own lens reports and the parent could wait on the files with `lym shuttle await`.
  The burler round audit treats any fork write as a hard error, though, and a fork writes its file a turn before its notification, so a late notification could still wake the parent.
  The transcript holds the notification itself, so the wait ends exactly when every result is queued for the parent.
- **The twill directive is included.**
  The board entry names reviewers and fixers, and the twill writer's lens review waits on its forks with the same sentence and the same cost.
  `twill-writer-fork.md` waits on one fork and is left as it is.
- **The Plan reviewer** already works during its forks and does not end its turn.
  It now ends its fork wait in one call too, so its trailing wake-ups go away as well.

## Verification

The tier0 gate is green, with two new tests:

- `TestOutstandingForks` reads a temporary project directory: an unanswered fork, a fork named by a queued notification, a shell beside a fork, and a missing transcript.
- `TestAwaitForks` runs the poll on a fake clock: ready at once, ready after the forks finish, the cap, a read error and a cancelled context.

`TestTranscriptHasUnmatchedLaunch_BashAndAgentLaunches` gains a fork-typed launch.

That every queued notification reaches the parent in one turn is Claude Code behaviour, read from transcripts, not tested here.
It is measured by the `fork-wait` column of `docs/research/review-turn-audit-2026-11-01/where-turns-go.sh` on the next tapestry runs.
