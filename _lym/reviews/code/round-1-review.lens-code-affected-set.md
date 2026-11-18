```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: what the diff's changes force outside the lines it touches.

**Per change, the affected set and whether it is covered**

1. **New refusal: a scope whose `memory.max` is not the cap is an `ErrMemoryCap` error.**
   - `internal/proc/memorycap.go:13`, the `ErrMemoryCap` doc ("whose command never ran under it"): covered.
   - `memorycap.go:29` and `:39`, the `RunMemoryCapped` and `RunMemoryCappedVia` docs ("a scope that never ran cmd included"): still true, since a refused scope never runs cmd. Covered.
   - `internal/proc/doc.go:75-76`: updated. Covered.
   - `internal/gatecli/tierrun.go:234` `tierMemoryCapRefusal` and `:451`, which check `errors.Is(runErr, proc.ErrMemoryCap)`: the new case is "a scope that never ran the command", and the way forward (the user manager hands the memory controller down, or remove `memory_mb`) fits it. Covered.
   - `internal/testcover/runner.go:330` returns the error as before. Covered.
2. **A holder process in the scope and in the shell's process group.**
   - `internal/gatecli/child.go:72` `killLeftovers` → `proc.KillTree`: not covered (finding 2).
   - `internal/gatecli/tierrun_memorycap_linux_test.go`: its background-sleep workaround for the race the diff fixes is now stale, and its hog is unbounded (finding 1).
   - PATTERN-child-processes "Reap children": the shell waits on the foreground subshell, which exits at once. The holder is reparented, and no lym process is a child subreaper (grep found none). It fits the double-fork-intermediate exception. Covered.
   - PATTERN-child-processes spawn logging: proc cannot log, as `doc.go:79` records. Covered.
3. **`watchForOOMKill` and its `unix.Poll` removed.** No reference is left outside `memorycap_linux.go`; grep found none. `errors` and `unix` are still used there. Covered.
4. **Script arguments (`wantMax` placed before `cmd.Path`) and the new fd 4 in `ExtraFiles`.** The only readers are inside `runMemoryCapped`. The `memorycap.go:28` doc already says cmd sets no `ExtraFiles`. Covered.
5. **Commit messages and departures.** None of `0b42877c..HEAD` carries a `Pattern-Departure:` trailer. No PATTERN changes are declared in the directive, and none landed. Covered.

**Findings**

1. **MEDIUM, design.** `internal/gatecli/tierrun_memorycap_linux_test.go:62-66`, `TestRunTierMemoryCapKillsARunawayCommandAndNamesTheCap`.
   - The tier command is `sleep 300 >/dev/null 2>&1 & cat /dev/zero | tail`. Only the cap bounds that hog's size, which breaks the decision record's Fix bullet that every probe and test hog is bounded in size and in time. It is the same `tail /dev/zero` shape that grew to 11.4 GB and hung the machine.
   - Its comment, "A sleep in the background keeps the scope alive after the shell exits. The kernel's OOM-kill count is then still readable.", works around the very race the diff's holder now closes, so it is stale.
   - Evidence: `tierrun_memorycap_linux_test.go:58-66` and `_lym/discussion/decision-record.md:22`.
   - Fix: make the hog the proc test binary's fixed-size hog, or a fixed-count `head -c` into `tail`, and drop the background sleep and its comment.
2. **LOW, consistency.** `internal/proc/memorycap_linux.go:62` together with `internal/gatecli/child.go:72` and `:85-91`.
   - The holder joins the shell's process group, and the `release` close happens only at `runMemoryCapped`'s return.
   - `runTiedChild` calls `killLeftovers(handle.Record())` right after. `killTreePlatform` matches any live process with that pgid that started after the record, so it can find the holder before it reads EOF and exits.
   - It then SIGKILLs the group and logs `gate test: killed what the shell left in its process group`. That reports a leftover the command never left, so the line no longer tells a real leftover from the holder.
   - Evidence: `tied_linux.go` `killTreePlatform` (pgid and start-time match), and `child.go:91`.
   - Fix: start the holder in its own process group, for example through `setsid` or `set -m`, or close `release` and wait for the holder's exit before returning.

Tool calls:
- Read `pattern/PATTERN-child-processes.md`: the holder is a new released process, and the manifest lacked that background file.
- Grepped for `ErrMemoryCap`, `memory.events` and the cap's doc phrases repo-wide: the manifest lacked readers outside proc.
- Grepped for subreapers, and read `gatecli/tierrun.go`'s cap refusal and handling and `gatecli/doc.go`'s cap lines: these are callers one hop out that read `ErrMemoryCap`.
- Grepped for `killLeftovers` users, and read `gatecli/tierrun_memorycap_linux_test.go`: the gate's own cap tests depend on the scope lifetime the diff changes.
