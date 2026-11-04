```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set.** For each change, the set I found and whether it is covered:

- **`runDobbyWith` gained a `reap func()` parameter.**
  - The set is `cli.go:387` (updated) and `dobbytrace_integration_test.go:59` (updated).
  - No other caller exists, so this is covered.
- **`proc.Reap` and `ReapInheritedChildren`, and the doc lines that describe them.**
  - The set is `proc/doc.go`, `beamcli/doc.go`, `orchengine/doc.go`, `selfexec/doc.go`, the doc comments of the changed spawn functions, and `PATTERN-spawn-observability.md`. All were updated.
  - The remaining stale comments are listed in the findings below. They are not covered.
- **New entry `PATTERN-reap-children`.** The set is `PATTERN.md` (the entry is added) and a `pattern/` background file (none is written or linked). Other entries without `(test)` also carry no background, so I do not flag the missing file. The entry's rule reaches spawns the diff does not cover; see finding 1. Not covered.
- **Zombie treated as gone in the registry.**
  - The set is `runproc` `List`/`Owners` and the readers that call them: `fabriccli/anchordeps.go`, `pairteardown/teardown.go`, `battencli/cancelcmd.go` and `tendercli/deps.go`. All of them inherit the new meaning through `List`/`Owners`.
  - The docs are updated in `runproc/doc.go` and `record.go`.
  - `PATTERN-run-process-owner.md` ("the read of a record prunes the ones whose process is gone") still reads correctly.
  - I found no other doc that describes a zombie as live. Covered.

**Findings**

1. **BLOCKING, design.** `reedengine/lifecycle.go:463-471` starts the tmux server (`cmd.Start()` inside `spawnSession`) and never waits on it or reaps it. The same shape is in `vscode/launch_linux.go:20-28` (`code` started and dropped).
   - The new `PATTERN-reap-children` entry in `PATTERN.md` says a starter waits on its child, "never only releasing it", with no short-lived-CLI exemption. These starts violate the entry as written, and the repo's rules make any violation of a listed constraint BLOCKING.
   - In a long-lived caller such as dobby, orch watch or a reed watchdog, each of these start paths leaves a zombie, the same defect as the goal. The diff either leaves them out or must narrow the entry's text.
   - I did not trace which processes reach `lifecycle.go`.

2. **LOW, consistency.** The doc comment at `fabricengine/spawn.go:66` still says "The child is started but never Waited". The diff changed `startDetachedPush` in the same file to call `proc.Reap`, so the comment is now false.

3. **LOW, consistency.** `adoptcli/deps.go:222` still says "The check is started and never waited for". The diff added `proc.Reap(command)` at line 251 of the same file.

4. **NIT, consistency.** `loomcli/start.go:574` still says "a detached Start with no Wait" about the spawn in `loomcli/drivewiring.go`. That spawn already waits through `go func() { _ = driveCmd.Wait() }()`, and the diff does not touch the file. This comment was stale before the diff, so it is optional.

5. **NIT, consistency.** `PATTERN-reap-children` says starters wait "by `cmd.Wait` or `proc.Reap`", but about five existing sites use a hand-rolled `go func() { _ = cmd.Wait() }()`. They are `orchcli/spawn.go`, `gatecli/detachfront.go`, `loomcli/drivewiring.go`, `driftengine/spawn.go` and `shedverbs/loopwait.go`. They comply with the entry. `proc.Reap` could replace the first two, but nothing requires it, so I raise no finding beyond this note.

**Tool calls**
1. One `grep`/`Bash` for `runDobbyWith(` callers, stale "never waited" and "Process.Release" phrases in the docs, and zombie/defunct mentions in the tender, fabric and pairteardown docs and `pattern/`. My context had none of the callers or doc phrases.
