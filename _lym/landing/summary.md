# Add `lym shuttle await-forks` so a forking role wakes once, not per fork

A role that spawns forks (a Discussion reviewer, Discussion fixer, Plan fixer, or a fix self-check) used to end its turn and wake once per fork, because each fork's result arrived as its own message.
Each wake-up was a turn with no call at the parent's full context.
On three tapestry runs that was 79 turns, 8% to 9% of the three roles' weight and 13% to 19% of their time (`docs/research/review-turn-audit-2026-11-01/report.md`).

After this change the parent spawns its forks in one message and then runs `lym shuttle await-forks` in the foreground.
The forks still run in parallel.
Every fork result reaches the parent together, in the turn after the verb answers `ready: true`.

## What changes

- New read-only verb `lym shuttle await-forks [--cap]` (`internal/shuttlecli/awaitforks.go`).
  It resolves the caller's run from its strand name and polls once a second until no fork the session launched is still running, or `--cap` elapses.
  It prints `ready: true`, or `ready: false` with the forks still running; a caller outside a shuttle strand, an unknown strand and a transcript read error exit 1.
  `--cap` defaults to `shuttleengine.DefaultAwaitCap`, under the prompt-cache lifetime, so the caller runs it with a Bash `timeout` of 300000 and keeps its cache warm.
- `Runner.AwaitForks` (`internal/shuttleengine/await.go`) holds the poll, over the runner's clock.
- New optional engine capability `OutstandingForkReader` (`internal/shuttleengine/engine.go`).
  `claudeengine` implements it by reading the session transcript: a fork launch that no later completion notification names is outstanding.
  An engine without the capability makes the verb fail, not report an empty list.
- The `Agent`/`Task` launch check in `claudeengine/background.go` now counts `subagent_type: "fork"` launches, which carry no `run_in_background` flag.
  The existing Stop-payload fallback shares this check, so `lym shuttle state` and the run-wait fallback now see fork launches too.
- The stencils `burler-cluster-fan`, `burler-fix-self-check` and `twill-directive` tell the agent to wait with the verb and never end its turn while a fork is outstanding.
  The re-run is bounded: after five `ready: false` answers in a row, or on a failure, the agent ends its turn and takes results as they arrive, as before.
- `docs/overview.md` and the package docs of `shuttleengine`, `claudeengine`, `burlerengine` and `twilldirective` describe the verb.
  `state.go` shares its task rendering with the new verb through `taskFields`.

## Design decisions

- **Wait in a verb, not by ending the turn.**
  The verb keeps the parent inside one tool call, so the harness queues every fork notification until the call returns and delivers them in one turn.
- **Transcript, not the Stop payload.**
  The verb runs mid-turn, so no Stop payload exists; Claude Code writes a notification that arrives during a tool call to the transcript at once, so the answer turns empty while the caller is still inside the call.
- **Unresolved launches count as outstanding.**
  A fork launch whose tool result is not yet in the transcript is reported with an empty ID, since its id is unknown but it is running.
  A launch whose result is present but carries no id is skipped, since nothing would ever name it.
- **Unreadable transcript is an error.**
  An empty answer would end the wait early and hand the parent a partial set.
- **Bounded re-runs.**
  A fork that never reports would keep the verb answering `ready: false`, so the stencils cap the re-runs and fall back to per-result wake-ups.

## To check by hand

Not run: the effect on wake-up counts.
Re-run the `fork-wait` measurement of `docs/research/review-turn-audit-2026-11-01` on a real run of the three roles, and confirm the forks still overlap in time.

Findings left open on the diff, to check by hand:

- `await-forks-verb-untested`: the `await-forks` verb in `internal/shuttlecli/awaitforks.go`, tested through a `strandResolver` seam in `cli_test.go` — confirm it covers the not-in-a-strand, unknown-strand, error and both ready answers.
- `runner-awaitforks-untested`: `Runner.AwaitForks` in `internal/shuttleengine/await.go`, tested in `await_test.go` — confirm it covers an unknown run, an engine without the capability and a read error.
- `outstanding-forks-drops-unresolved-launch`: `OutstandingForks` and `unmatchedLaunches` in `internal/shuttleengine/claudeengine/background.go` — confirm a fork launch with no recorded tool result stays outstanding, and that the shared launch check changes nothing for the Stop-payload fallback beyond counting forks.
