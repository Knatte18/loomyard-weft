```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fixer carried out as proposed (F1, F2, F4–F7).

1. **MEDIUM** — `scopeStartScript` in `internal/proc/memorycap_linux.go`, the F5 change `setsid sh -c 'read -r line' <&4 &`.
   - **Problem:** the holder now depends on `setsid` being found through PATH inside the scope. The earlier holder used only the `read` builtin.
   - **Evidence:**
     - The scope inherits the caller's `cmd.Env`.
     - Other tests build restricted environments, for example `[]string{"PATH=" + os.Getenv("PATH")}` in `tierrun_memorycap_linux_test.go:37`.
     - The subshell sends the holder's stderr to `/dev/null`.
     - If `setsid` is missing or not on PATH, the holder exits at once and fd 4 closes. The command still runs.
     - Nothing reports the failure, and the `memory.events` read races the scope's removal again, which is the bug this round closes.
   - **Test gap:** the new test `TestRunMemoryCapped_TheHolderEndsOnceACappedRunWithNoKillReturns` passes with no holder at all, because it only checks that the scope disappears. No test pins that the holder is alive after the command ends.

2. **LOW** — the F7 test `TestRunMemoryCapped_TheHolderIsNeverAChildOfTheCommand`.
   - **Problem:** the command reads `/proc/self/task/*/children`, which exists only with `CONFIG_PROC_CHILDREN`. Without it the glob stays literal and `cat` fails, so the test fails on that kernel instead of skipping. The skip guard checks only `MemoryCapCoversTree`.
   - **Evidence:** `memorycap_linux_test.go`, the new test's `exec.Command("sh", "-c", "exec cat /proc/self/task/*/children")`.

3. **LOW** — the F4 test `TestRunMemoryCapped_TheHolderEndsOnceACappedRunWithNoKillReturns`.
   - **Problem:** it treats a vanished scope directory as proof that the holder ended. A scope that never lived beyond the command's exit would also pass, so it does not separate "holder ended" from "holder never started".
   - **Evidence:** this is the same gap as finding 1.

F1, F2, F6 and the F5 doc text: no findings.
- F2: `head -c 268435456 /dev/zero | tail` still exceeds the 64 MB cap and the shell exits non-zero when `tail` is OOM-killed.
- F6: `wantMax` rounds to 134217728, but the unrounded value differs, so a broken round-down would fail the new row.

Further tool calls: none, my context already held all changed files.
