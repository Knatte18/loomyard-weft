# Card 1 — shuttle-background-tasks

**Create:**
- `internal/shuttleengine#BackgroundTask`

**Rename:**
- `internal/shuttleengine/claudeengine#hasOutstandingBackgroundWork` -> `internal/shuttleengine/claudeengine#outstandingBackgroundTasks`

**Edit:**
- `internal/shuttleengine#Event`
- `internal/shuttleengine/claudeengine#transcriptHasUnmatchedLaunch`
- `internal/shuttleengine/claudeengine#Claude.ParseEvents`
- `internal/shuttleengine/claudeengine/background_test.go`
- `internal/shuttleengine/claudeengine/events_test.go`
- `internal/shuttleengine/claudeengine/doc.go`

**Intent:**
D1's first half: `claudeengine` reports the outstanding background tasks of a waiting turn end as a provider-neutral list, so shuttle's wait loop can tell a fork from a shell.

In `internal/shuttleengine/engine.go`, add `BackgroundTask` with three fields: `Kind`, `ID` and `Label`.
`Kind` is a string-backed `BackgroundKind` with two constants, `BackgroundFork` ("fork", an Agent or Task subagent) and `BackgroundShell` ("shell", a backgrounded Bash or a Monitor).
`Label` is the shell's command or the Monitor's description, empty for a fork.
`Event` gains `Outstanding []BackgroundTask`, set only on an `EventWaiting`; its doc comment says so.

In `claudeengine/background.go`, rename `hasOutstandingBackgroundWork` to `outstandingBackgroundTasks`, returning `[]shuttleengine.BackgroundTask` instead of a bool.
Both signals feed the list, deduplicated by id:
a `background_tasks[]` entry whose `status` is "running" (type "subagent" is a fork, any other type is a shell; the label is the entry's `command`, else its `description`, else its id, per R2),
and every transcript launch no later `<task-notification>` names (`transcriptHasUnmatchedLaunch` returns those launches with kind and label instead of a bool: Agent/Task launches are forks, Bash and Monitor launches are shells, the label is the launch input's `command` or `description`).
The transcript signal is no longer consulted only when the payload shows nothing running, since a running shell entry and an unmatched fork launch can both be outstanding.
Replace the file comment's "only observed type subagent entries" claim, which the `board-model` run's shell entry disproved, with what the code now reads.
An unreadable transcript still counts as nothing outstanding from that signal.

`Claude.ParseEvents` (`events.go`) classifies a Stop line `EventWaiting` when the list is non-empty and sets `Event.Outstanding` from it.
All Claude payload-shape knowledge stays in `claudeengine` (PATTERN-shuttle-provider-seam).

Tests: a Stop payload with a running `background_tasks[]` shell entry in the `board-model` shape yields one shell task with its command as label;
a transcript-only backgrounded Bash with no notification yields a shell task labelled with its command;
an unmatched Agent launch yields a fork task;
a payload entry and a transcript launch sharing an id yield one task;
`ParseEvents` carries the list on the `EventWaiting` it emits.
Update `claudeengine/doc.go` where it describes the waiting signal.

**ImpactSummary:** claudeengine's waiting-detection now returns a task list; `Event` gains one field every existing reader ignores.
