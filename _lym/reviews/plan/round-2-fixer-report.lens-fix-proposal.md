```yaml
lens: fix-proposal
outcome: reported
```

Task: as the fix-proposal lens, challenge the reviewer's proposals that the fixer carried out (F1 in card 8 and the F2 ruling in cards 7 and 1).

**MEDIUM: card 8's marker row compares a measured real time to a bound a loaded machine can reach**

- **Changed text:** `_lym/plan/08-plugin-session-start-contexts.md`, the `TestRunPluginHook` bullet in Tests: "`(sleep 2; touch $MARKER) & wait` … the test waits 3 seconds and finds through `os.Stat` that the marker never appeared".
- **Summary:** the row decides that the group kill worked from a race between a 2-second sleep and a 3-second wait, which the pattern bars. A pid would tell the two paths apart directly.
- **Evidence:** `pattern/PATTERN-test-speed.md:178-182` says "No test compares a measured real time to a bound a loaded machine can reach." It also says: "Where an outcome already tells the two paths apart, such as an exit code, an error or a pid, the test asserts that outcome instead of a time."
- **Concrete case:** the group kill is missing and the machine is loaded, so the surviving subshell's `touch` lands later than 3 seconds after the error. `os.Stat` finds no marker and the row passes. The row is meant to pin the group kill, and here it passes without it.
- **Alternative:** the command backgrounds a long-lived child and writes its pid, for example `sleep 120 & echo $! > $PIDFILE; wait`. After the timeout error, the test reads the pid and polls `syscall.Kill(pid, 0)` until it returns `ESRCH`, under a 60-second deadline. A surviving child outlasts that deadline by a wide margin, so the outcome no longer depends on load.
  - This needs the pid file to exist before the 100ms timeout fires, and under load that write races the timeout. A longer `Timeout` for this row, still far below the 60-second deadline, closes that race.

**No finding on the other proposals:**
- The `sleep 120; true` row with a 60-second deadline fits the pattern's "generous deadline" clause.
- The F2 ruling's directory-at-path input is sound. `os.ReadFile` on a directory fails with EISDIR for root too, and that error is not `fs.ErrNotExist`. That matches card 7's absent-file row and card 1's "missing import stays as written" row, and nothing else in either card conflicts with it.

Tool calls:
- grep of `pattern/PATTERN-test-speed.md`, "Time is injectable" section: my context held only the review's paraphrase of the rule, not its text.
