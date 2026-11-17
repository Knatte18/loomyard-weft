```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff `bd707e16..HEAD`. For each function and branch the diff adds or changes, I ran one concrete case through it.

**Functions and branches, the case run, and the verdict**

- **`scopeStartScript`, normal start.** `cat` writes the scope's own `/proc/self/cgroup`, `0::/user.slice/…/run-pX.scope`, to fd 3. The shell waits for `cat`, closes fd 3 and execs the command, so the command and its children never hold fd 3. Correct.
- **`scopeStartScript`, `cat` not found.** The env's `PATH` lacks `cat`, so the shell writes nothing and still execs the command. `scopeCgroup` then returns `ok=false` for every run, including one that exits 0. `runMemoryCapped` reports "the scope did not run the command". **Incorrect, finding 3.** The old `echo` was a shell builtin and could not miss.
- **`scopeCgroup`, writer is `/bin/false` (scope never ran).** fd 3 closes when `false` exits. `ReadAll` sees EOF with an empty listing, and `cgroupV2Path` returns `ok=false`. Correct, and the existing starter test pins it.
- **`scopeCgroup`, listing already in the pipe and all write ends closed after `Wait`.** `ReadAll` returns the whole listing at once; the listing is a few bytes, far below the pipe buffer. The deadline never matters. Correct.
- **`scopeCgroup`, a held write end or a partial listing at the deadline.** No process holds fd 3 after the shell's `exec 3>&-`. If one somehow did, `ReadAll` would return a timeout error and `ok=false`, the same as the old one-byte read. No stall. Correct.
- **`scopeCgroup`, listing without a `0::` line (cgroup v1).** `ok=false`, but the probe already requires a v2 path, so this is unreachable. Correct.
- **`runMemoryCapped`, cancel path.** `cancelled=true` returns before `scopeCgroup` and `killScope`. Processes outside the group stay in the scope, as they did before the diff, because no OOM event was involved. No regression.
- **`runMemoryCapped`, exit 0 or a non-SIGKILL exit.** It returns `false, err` without `killScope`. Correct for the main-process case. Finding 1 covers the OOM victim being a child.
- **`runMemoryCapped`, kernel SIGKILLs the main process at the cap.** With `OOMPolicy=continue` the kernel still picks a victim with SIGKILL; the policy only stops systemd from failing or stopping the unit. `killed=true`, then `killScope` runs. Correct.
- **`runMemoryCapped`, main killed and a leftover child still holds the stdout pipe.** `cmd.Wait` cannot return until `WaitDelay` (testcover's `tc.Grace`) closes the pipes. Only then does `killScope` run. Before the diff, the `stop` policy ended the leftovers at once. **Delay, finding 2.**
- **`runMemoryCapped`, `killScope` fails.** `errors.Join(ExitError, killErr)` and `killed=true`. `errors.As` still finds the `*exec.ExitError`, and testcover's `RunAlone` keys on `killed`, so the result stays `MemoryCapped`. Only the error text grows. Correct.
- **`killScope`, scope already collected.** The directory is gone, `open(O_CREAT)` returns ENOENT, and that is no error. Correct.
- **`killScope`, cgroup removed while being written.** ENODEV, no error. Correct.
- **`killScope`, kernel older than 5.14 (no `cgroup.kill`), directory still present.** `os.WriteFile` opens with `O_CREAT|O_TRUNC`, and kernfs refuses to create the file (EPERM/EACCES). The error is joined into every cap kill, and leftovers are not killed. **Minor, finding 4.**
- **`memoryScopeArgs`.** It adds `-p OOMPolicy=continue` before `--`, and the argument order is valid. The probe uses the same arguments and caps nothing that is killed. Correct.

**Findings**

1. **MEDIUM, design.** `internal/proc/memorycap_linux.go:101` and `:155-158`: an OOM kill of a process other than the main one now leaves the rest of the scope running, and nothing ends it.
   - Before, the default `OOMPolicy=stop` stopped the whole scope on any OOM kill inside it (`/etc/systemd/system.conf:80`, `#DefaultOOMPolicy=stop`).
   - With `continue`, the kernel may pick a child as the victim: it chooses the largest process in the memcg, for example a hog subprocess a test binary spawns.
   - The main process then exits without SIGKILL, so `runMemoryCapped` returns at `:157` without calling `killScope`. The child's siblings outlive the call.
   - This contradicts the new doc's promise in `memorycap.go:11`, "Every process the command left in the scope is then killed", which the new code keeps only for a main-process kill.
2. **LOW, design.** `internal/proc/memorycap_linux.go:147,159`: `killScope` runs only after `cmd.Wait` returns.
   - A leftover process that inherited the command's stdout or stderr pipe keeps `Wait` blocked.
   - testcover passes a `bytes.Buffer` (`internal/testcover/runner.go:176`), so the block lasts until `WaitDelay` (`tc.Grace`, `runner.go:370`) expires, and forever for a caller with no `WaitDelay`.
   - Before the diff, the `stop` policy killed those holders at once. Each cap kill whose binary left such a child now costs up to `Grace` before cleanup.
3. **LOW, design.** `internal/proc/memorycap_linux.go:47`: `scopeStartScript` now runs external `cat`, found through the caller's `PATH`, where it used to run the builtin `echo`.
   - A `cmd.Env` whose `PATH` lacks `cat` makes every capped run, successful ones included, fail with "the scope did not run the command" (`:151-153`).
   - The literal `/proc/self/cgroup` also duplicates the `selfCgroupFile` constant (`internal/proc/tasklimit.go:16`).
4. **LOW, design.** `internal/proc/memorycap_linux.go:182-183`: `os.WriteFile` opens with `O_CREAT|O_TRUNC`.
   - On a kernel without `cgroup.kill` (before 5.14), the open fails with a permission error rather than ENOENT, so every cap kill gets a spurious joined error while the scope directory still exists.
   - Leftovers go unkilled there, and nothing degrades gracefully.

**Tool calls**

- Grepped `internal/testcover/runner.go` for `WaitDelay`, `Env` and `CommandContext`, and `/etc/systemd/system.conf` for `DefaultOOMPolicy`: the manifest lacked the capped caller's `WaitDelay` and env setup, and the host's default OOM policy.
- Read `internal/testcover/runner.go` lines 160-200 and 355-380: the manifest lacked `RunAlone`'s use of `killed` and the `command` builder that sets `WaitDelay` and the group kill.
