```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over `runMemoryCapped` and `scopeStartScript` in `0b42877c..HEAD`.

**Per function or branch: the case I ran, and whether it is correct**

- `runMemoryCapped`, creating the holder pipe fails. `marker` closes in place and `started` closes by defer, so no descriptor leaks. Correct.
- `wantMax` with a limit that is not a page multiple: 1000000 gives `1000000 &^ 4095` = 999424. The kernel stores `bytes/PAGE_SIZE` pages and shows `pages*PAGE_SIZE` = 999424. With limit 100 it gives 0 and the kernel shows `0`. `testMemoryCap` is 128 MiB and maps to itself. Correct: a legitimate cap is not refused.
- `start` returns an error. Both `marker` and `holderEnd` are closed, and the deferred `release` closes. If the process started and the start then failed, the holder sees end of file and exits. Correct.
- Cancel path. `wait` returns, the marker is read with the 1 s bound, and the call returns early with `release` closed by defer. A group kill reaches the holder because it shares the shell's process group. A cancel that kills only the child leaves the holder until return, which follows at once. Correct.
- `wait` with `WaitDelay` (the `runTiedChild` caller). The holder's stdio is `/dev/null` and its fd 3 was closed before the fork, so it pins neither the output pipes nor the marker pipe. Correct.
- `ranUnderCap` false:
  - `/bin/false` exits 1 and writes no marker, so the error wraps `ErrMemoryCap` and the `%v` keeps `*exec.ExitError` out of the chain.
  - A mismatch exits 125 with no marker, giving the same error.
  - The command itself exiting 125 after the marker counts as `ranUnderCap`, so it returns the command's own `ExitError`.

  Correct.
- `ProcessState == nil` ahead of the `ranUnderCap` check: unchanged in order. Correct.
- `oomKilled` with the command already gone (the test row whose start returns after the command has ended). The holder keeps the scope, so `memory.events` is readable, and `killScope` then ends the holder. Correct.
- No kill. The read gives false, the call returns, and the deferred `release.Close` makes the holder's `read` hit end of file and exit. The scope empties and is collected. Correct.
- `scopeStartScript`, which I ran under dash and bash during reading:
  - A mismatch prints both values to stderr and exits 125.
  - A match writes `0::<scope>` to fd 3 and runs `printf '%s|' a 'b c' ''` with its arguments intact, empty one included.
  - With no `0::` line, `$scope` is empty, the read of `/sys/fs/cgroup/memory.max` fails and the script exits 125.
  - `A && B || { …; exit 125; }` runs the block when either A or B fails.
  - `$1` and `"$@"` are quoted, and `shift` removes only `wantMax`.

  Correct.
- Where the holder is reparented: nothing in the repo sets a child subreaper (grep below). The orphaned holder therefore goes to the systemd user manager or init, which reaps it, and the Go caller collects no zombies. Correct.
- The subshell `( … & )` as a child of the command. The subshell is forked before `exec "$@"`, so after the exec it is a child of the command's process: a zombie, or about to become one. Not correct as the comment states it; see the finding below.

**Findings**

1. LOW · `design` · `internal/proc/memorycap_linux.go:53`, `:62`
   - Summary: the comment says "The holder leaves the shell's tree at once, so the command never sees it as a child of its own". But the subshell that forks the holder is a child of the shell, and the shell's pid becomes the command through `exec "$@"`. So the command inherits a child it never started, a zombie once the subshell exits.
   - Who it affects: a command that reaps any child, such as `wait(NULL)` or `waitpid(-1, …)` in a C program, or a `wait -n` shell, can collect the subshell's exit status in place of its own child's. The current callers, `go test`, Go test binaries and dash, wait by pid or ignore unknown pids, so none is affected today.
   - Fix: let the comment say what holds, that the holder itself is never the command's child and only a short-lived subshell is. Or have the subshell reaped before the exec, for example by running it in the foreground (`( … & )` returns once the background job is forked) and letting the shell `wait` for it before `exec`.
   - Evidence: `memorycap_linux.go:62` (`( exec </dev/null >/dev/null 2>&1; read -r line <&4 & ); exec 4<&-; shift; exec "$@"`).

2. LOW · `design` · `internal/proc/memorycap_linux.go:62`
   - Summary: the holder shares the command's process group, so a command that signals its own group, such as `kill 0` or `trap 'kill 0' EXIT`, kills the holder. If that happens before the command ends, the empty scope can be removed before `scopeHadOOMKill` reads it, and a kill at the cap goes unreported: the same race the diff closes. Cancels are unaffected because they return before the read.
   - Fix: start the holder in a process group of its own. A non-interactive shell's background job stays in the shell's group, and `setsid` has the same reach-out effect. Or record this limit in the doc comment.
   - Evidence: `memorycap_linux.go:62`, and `tied_linux.go` `startPlatform` (`Setpgid` puts the whole tree, holder included, in one group).

No other correctness defects in the changed functions.

Tool calls:
- grep for `SUBREAPER`/`Subreaper` in production Go: to learn where an orphaned holder is reparented and whether it can leave zombies with the Go caller. The manifest did not cover process reparenting.
