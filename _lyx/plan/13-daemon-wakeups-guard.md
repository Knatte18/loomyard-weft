# Card 13 — daemon-wakeups-guard

**Create:**
- `pattern/PATTERN-daemon-wakeups.md`
- `cmd/lyx/daemonwakeups_test.go`

**Edit:**
- `PATTERN.md`
- `internal/orchengine#watchLockRetryInterval`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** The rule and its guard (D1, D2).
`PATTERN.md` gains a section "Daemons and waits" between "Agents and prompts" and "Packages" holding the one-line `PATTERN-daemon-wakeups` entry from D1, marked `(test)`, linking its background file, within the format checker's line length.
`pattern/PATTERN-daemon-wakeups.md` states what counts as an unbounded wait (the detached daemons `lyx reed watchdog` and `lyx orch watch`, batten's Run-Shed wait, shuttle's `Wait`, webster's long poll, `lyx shed status --watch`), what does not (a bounded wait that ends within a minute), the one-second floor for any periodic wake, the backoff rule for a wake that spawns a process or captures a pane (doubling from its base to a named ceiling while nothing changes, reset on a change or an event), that a configured interval below the floor is floored with one Warn and never refused, that `internal/fswatch` is the one file-event source and a tmux hook touching a file is how tmux state becomes a file event, and the guard's bound: the scan sees literal shapes only, so a computed sub-second period or a `time.Sleep` loop in an unlisted file is caught by review.
`cmd/lyx/daemonwakeups_test.go` holds one untagged test marked `//lyx:guard`, in the manner of `cmd/lyx/tierpurity_test.go`: a declared list of the unbounded-wait files (`internal/reedengine/watchloop.go`, `internal/reedengine/namerepair.go`, `internal/reedcli/watchdog.go`, `internal/orchengine/watchloop.go`, `internal/battenshed/innerrun.go`, `internal/battenshed/primelockwait.go`, `internal/shuttleengine/wait.go`, `internal/websterengine/poll.go`, `internal/shedverbs/status.go`), each of which must exist.
Over each declared file it parses the AST and fails on a sub-second duration literal (`N * time.Millisecond` with N below 1000, any `time.Microsecond` or `time.Nanosecond` multiple) and on an integer constant whose name ends in `MS` or `Ms` with a value below 1000, unless the source line carries `//lyx:one-shot <reason>`; the duration arithmetic reuses `cmd/lyx/tiersleep_test.go`'s scale helpers.
A bare `//lyx:one-shot` with no reason does not let its line through, and the failure says a reason is required.
Bound: the annotation is an escape hatch that lets any sub-second literal through on the line that carries it, so the guard enforces announcement, not absence (D2's Bound); what bounds it is that the reason sits on the line itself, where review of the commit reads it.
Over every production `.go` file under `internal/` and `cmd/` (`scankit.Walk` with the `Prod` filter and a vacuous-scan floor) it fails on a `time.NewTicker` or `time.Tick` call outside the declared list, naming the file and the way forward: list the loop or explain it in the same commit.
Subtests of the same guard test prove the detector fires on crafted snippets for each literal shape and on a bare annotation, and passes a line annotated with a reason, as `TestFindLongLiteralSleep_DetectsAllArgumentForms` does for its scan.
`orchengine.watchLockRetryInterval` carries `//lyx:one-shot bounded lock retry of watchLockAttempts` on its line, the one sub-second literal left in a declared file after the earlier cards.
`cmd/lyx/testdata/test-budget.yaml` raises `cmd/lyx` by the one test.

**ImpactSummary:** A new repository-scanning guard and one annotated constant; the guard passes on the tree as the earlier cards leave it, and no behaviour changes.
