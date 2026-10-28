# Support log — dobby-task-limit-hold

## Interview

A darn run holds no interview: no operator answered, and every pick below is a self-pick recorded in the question ledger.

Read the board entry, `lym board list` (no `Decision for dobby-task-limit-hold:` note), `lym loom peers` (no peer touches `internal/dobbyengine` or `internal/proc`), commit 88291c54 that left this half of #885 unbuilt, `internal/dobbyengine` (start step, config, view, loop), `internal/proc/doc.go` and the dobby wiring in `internal/beamcli/dobbydeps.go`.
On this machine the dobby's ancestry is `user.slice/user-1000.slice/user@1000.service/app.slice/<scope>`, and only `user-1000.slice` sets a numeric `pids.max`.

## Rejected alternatives

- Reading `user-<uid>.slice` by a computed path: ties the reader to systemd's layout and misses a tighter limit lower in the tree.
- A margin as a percentage of `pids.max`: the incident's cost is per run, an absolute count of tasks, and a fixed count stays meaningful when the operator raises the limit.
- Recording a held task as a refused start with notice and backoff: the hold is no refusal, and a backoff of up to an hour would delay the start long after the headroom returned.
- An orch notice per hold: the board entry asks for a log line; a notice can be added later if holds go unseen.
- Showing the hold as a waiting reason in the dobby view: the view is built in another process, which would need its own reading; left out to keep this change to the start.
- Holding continue and full passes of runs already started: a held pass stalls a run that already holds its processes and frees none.

## Review rounds

_No rounds yet._

## Question ledger

- Which cgroup level is "the user slice"? Self-pick: the tightest level of the dobby's own ancestry (D1).
- What does an unreadable cgroup do? Self-pick: no hold, as the board entry says (D2).
- Where does the margin live, and its default? Self-pick: beam.yaml `task_headroom`, 500, 0 off (D3).
- Is the check once per tick or per start? Self-pick: per start, as the reading is two file reads per level (D4).
- How is the hold logged? Self-pick: one line at the start and one at the end of an episode (D5).
