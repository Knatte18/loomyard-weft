# Card 11 — adopt-pid-reuse

**Edit:**
- `internal/proc/proc_linux.go`
- `internal/proc/proc_windows.go`
- `internal/proc/proc_linux_test.go`
- `internal/shuttleengine/claudeengine/resume.go`
- `internal/shuttleengine/claudeengine/resume_test.go`
- `internal/shuttleengine/claudeengine/doc.go`

**Intent:**
Stop a reused pid from refusing an adopt (D15).

`internal/proc` gains `StartTime(pid int) (string, bool)`, the process start time in the form Claude's session registry records it as `procStart`.
On Linux it is field 22 (`starttime`, clock ticks since boot) of `/proc/<pid>/stat`, parsed after the last `)` so a command name with spaces or parentheses cannot shift the fields; the implementer confirms the form against a live registry entry before relying on it.
On Windows it reports unreadable (false), which keeps today's fail-closed behaviour there.

In `resume.go`, `registryEntry` gains `ProcStart string` (`json:"procStart"`), and `checkResumable` takes a start-time reader beside `alive`.
A registry entry naming the session refuses the adopt only when its pid is alive and the live process's start time matches the entry's `procStart`.
When either start time cannot be read (the reader fails or the entry has none), a live pid still refuses, and the refusal says the pid could not be proven reused.
Bound: the check lets an adopt through only on a positive start-time mismatch, which proves the pid was reused; an unreadable start time fails closed so two processes never drive one session.
Grep for the existing live-holder refusal text: the smoke test `internal/orchcli/smoke_live_test.go` asserts `live process`, so keep that phrase in both refusal variants.

Tests: `resume_test.go` covers matching start time refuses, mismatching start time lets the adopt through, unreadable live start time refuses with the "could not be proven reused" wording, and an entry with no `procStart` refuses the same way.
`proc_linux_test.go` reads the test process's own start time and gets a non-empty value, and gets false for a pid that does not exist.
`internal/shuttleengine/claudeengine/doc.go` states the rule where it describes the resume check.

**ImpactSummary:** Adds a start-time comparison to the Claude resume check and one function to internal/proc on both platforms; a live, unproven holder still refuses.
