# Card 2 — Reed reads a pane's age from /proc

**Create:**
- `internal/reedengine#processAge`
- `internal/reedengine#paneProcessAge`
- `internal/reedengine#TestProcessAge`

**Uses:**
- `internal/reedengine#parseStat`

**Intent:**
D1's launch grace needs a pane's age: the elapsed time of the pane's first process, the `#{pane_pid}` reed's listings already read, so a `Status` read spawns no process and no launch time goes on the strand (`PATTERN-tolerant-records`).

`processAge` is the pure helper in `internal/reedengine/proctree.go`, beside `parseStat`, so it stays testable on every host:
it reads the system uptime in seconds from the first field of `uptime` (the content of `/proc/uptime`, a decimal such as `12345.67`), reads the process start time from field 22 of `stat` (the content of `/proc/<pid>/stat`, clock ticks since boot), anchored on the last `)` exactly as `parseStat` does so a comm with spaces or parentheses parses, and returns the uptime less `start / ticksPerSecond` as a `time.Duration`.
It returns an error for an empty or non-numeric uptime, a stat with no closing paren or fewer than 20 fields after it, a non-numeric start, a `ticksPerSecond` of 0 or below, and a start later than the uptime (a negative age).

`paneProcessAge` is the platform probe.
On Linux (`proctree_linux.go`) it reads `/proc/uptime` and `/proc/<pid>/stat` with `os.ReadFile`, as the file's other probes read `/proc`, and passes the clock tick rate `sysconf(_SC_CLK_TCK)` returns, read through cgo (`import "C"` with `#include <unistd.h>`), which the `lyx` build already needs (`PATTERN-quarry-cgo`).
It answers false for a pid of 0 or below, an unreadable file and any `processAge` error, since an age that cannot be read counts as inside the grace (D1, Bound) and so can never make a removal kill a pane.
On Windows (`proctree_windows.go`) it answers 0 and false: condition 3 is skipped there (D1), so nothing reads it, and the file keeps the package building.

`TestProcessAge` is a table over `processAge`: a stat whose comm holds spaces and parentheses answering the expected age at 100 ticks per second, and each error case above.
It also drives the platform probe, so the cgo tick rate and the `/proc` reads ship with a row: on Linux, `paneProcessAge(os.Getpid())` answers true with an age of at least 0 and under an hour, and `paneProcessAge(0)` and `paneProcessAge(-1)` answer false; on Windows every row expects false.
`TestParseStat` is not extended, since `parseStat` does not change.

**Commit:** 2: reed reads a pane's age from /proc
