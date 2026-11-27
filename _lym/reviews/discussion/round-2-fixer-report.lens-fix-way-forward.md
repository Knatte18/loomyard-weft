```yaml
lens: fix-way-forward
outcome: reported
```

Task: check the ways forward this fix introduced (fix-way-forward lens).

1. **MEDIUM.** The changed text is Open risks, the new KWin-facts entry: "The implementing run's first step probes them … It may run the read-only probes itself".
   - **Summary:** that step may be refused in the state the record names. A read-only probe still loads a script into KWin through `qdbus6` over the user session bus, and lym sessions guard against that bus.
   - **Evidence:** this session's harness guard refuses `busctl --user`, `gdbus` and a `dbus-send` without `--system`, because "the user session bus they reach starts processes outside lym". It also says "When the work needs the session bus, say so to the operator or your parent". `qdbus6 org.kde.KWin /Scripting loadScript` reaches the same bus.
   - **Gap:** the entry gives no fallback for the implementing run when its probe is refused, such as asking the operator to run it. So the step "it may run itself" can dead-end.

2. **MEDIUM.** The changed text is the same Open risks entry: "A fact a probe contradicts reopens the decisions that rest on it before any code relies on them."
   - **Summary:** the way forward names no actor and no route. An implementing run cannot reopen a decision record by itself.
   - **Evidence:** the entry gives no step such as "stop and raise to the parent with the probe's output".
   - **Gap:** the contradiction case leads nowhere that makes progress.

3. **LOW.** The changed text is D12: "A child ended by SIGTERM runs no `window_command`, and it ends one already running before it exits."
   - **Summary:** the step needs a cancel path that the window-command runner lacks, and the record does not say the runner gains one.
   - **Evidence:** in `internal/desktophook/window.go`, `runWindowCommand` builds its context from `context.Background()` with only the 30 s timeout, and `AfterWindow(dirs, dir)` takes no context. `exec.CommandContext` kills only the shell process, not the shell's children.
   - **Gap:** when the opening verb runs outside a run, the child is not registered with `runproc`, so nothing else reaps those children. The step can be carried out only if the plan adds a context parameter or a process-group kill, and the record names neither.

4. **LOW.** The changed text is D8 and D10's return to the parent wait: "releases the lock and returns to that wait", with the bound "runs from the child's first entry into the wait and covers every return to it".
   - **Summary:** the parent wait is bounded, but each return also takes a new lock acquire under D10's 120 s timer. The record states no bound on the child's total time before `window_command` runs.
   - **Evidence:** D10 says "A timer ends the wait after 120 seconds". D8 bounds only the 180 s parent wait.
   - **Gap:** a child that returns more than once can spend 180 s plus 120 s per acquire, with no stated cap.

Tool calls:
- Read `internal/desktophook/window.go` lines 1–33 and 57–100, to check whether the window-command runner can be cancelled. My context held only its `productionTiming` lines.
