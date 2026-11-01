```yaml
lens: fix-claims
outcome: reported
```

Task: check that each claim the fix added or rewrote holds against the code at HEAD (fix-claims lens).

**1. MEDIUM: a parent-death signal does not kill a tier's whole tree**
- **Changed text:** two sentences.
  - D3: "A `lym gate test` runner's slots free with its OS file locks, and its tiers die through their `proc.StartTied` tie"
  - D6: "a `--wait` gate child among them, whose own tiers die through their `proc.StartTied` tie"
- **Summary:** `proc.StartTied` sends its parent-death SIGKILL only to the tier's shell. The tier's descendants, such as the test binaries `go test` starts, live on after the runner or gate child is killed.
- **Evidence:**
  - `internal/proc/tied_linux.go:18-19` sets `Setpgid` and `Pdeathsig = SIGKILL` on the started child only. A parent-death signal reaches that one process, never its process group.
  - The group kill (`killPlatform`, `syscall.Kill(-PGID, SIGKILL)`) runs only through `Tied.Kill`. That happens on a cancel (`internal/gatecli/child.go:24-30`), never on the parent's death.
  - So a SIGKILLed runner leaves each running tier's process group alive until it ends by itself.
  - Both sentences state a tie that the code does not provide for the whole tier tree.

**2. LOW: D5's list of report totals misses the launch totals**
- **Changed text:** D5: "`lym gate report` leaves a record carrying `Abandoned` out of its run counts and its wait, wall and build totals."
- **Summary:** the report also totals launch time and package time. The sentence's list leaves them out, while the D5 criterion and the Constraint say "its totals".
- **Evidence:**
  - The `reportGroup` in `internal/gatecli/report.go` also holds `launchSeconds`, `packageSeconds` and `hasLaunches`, summed from `record.Launches`.
  - An abandoned record's launch list would either still count or the sentence is incomplete.
  - The acceptance criterion ("left out of its totals") and the PATTERN-tolerant-records line ("stay out of its totals") say something broader than D5's own list.

These claims hold:
- `proc.StartTime` and `proc.CreationTime` per platform.
- Every record of a call goes through `execution.writeRecord`: the build step, `reportBlocked`, and `report`, which `reportCarried` also calls.
- The pre-tier phase is the config load, the scope computation and the guard scan.
- The build and the tiers start through `runTiedChild` and `StartTied`.
- On Windows, `verifyrun`'s shell sets no breakaway flag, so it stays in a job the runner holds.
- `proc.KillTree` reaches a group after its leader dies.
- `card.ID()` covers both accepted forms of a card id.
- The 60 s floor fits the lock wait plus the 30 s claim wait.

Tool calls:
- Read `internal/proc/procgroup_windows.go` flags, the `writeRecord`/`WriteRunRecord` callers, and the tier and build spawn sites, to check the Windows job inheritance and the `writeRecord` coverage. My context lacked all three.
- Read the build and tier spawn code in `build.go` and `tierrun.go`, plus `procgroup_windows.go`, to check how tiers and the build are tied and whether `verifyrun` breaks away on Windows. My context lacked these files.
- Read `run.go:50-95` and `runTiedChild` in `child.go`, to check the pre-tier phases and what kills a tied child's group. My context lacked both.
