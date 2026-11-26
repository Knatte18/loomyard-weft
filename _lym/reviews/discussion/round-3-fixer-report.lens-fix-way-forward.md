```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that every refusal, next step, row and test the round-3 fix added can be reached and leads somewhere useful. I found two MEDIUM issues and two LOW ones.

1. **MEDIUM — the D6 attach wait has no way out when the supervisor is dead.**
   - Changed text: "`lym reed attach` waits on the client's process; the supervisor watches the client's pid and, once it exits, ends the attach scope and answers the waiting attach, which only then exits, so no process of the scope holds the terminal when a host process gets it back."
   - Problem: the step relies on a live supervisor, but D4 and D3 both describe the supervisor dying with the tree still running. In that state nothing answers the waiting attach, so it either never returns or returns while orphans of the scope still hold the terminal.
   - Missing: what the waiter does when the supervisor's socket closes, for example end the scope itself or refuse with `lym area stop`. No acceptance criterion covers this case either.

2. **MEDIUM — D6 moves the machine record and names no migration.**
   - Changed text: "The hub directory, `<per-user lym state dir>/hubs/`, holds the machine record `hubs.json`, its lock `hubs.lock` …"
   - Problem: the record moves from `<state dir>/hubs.json` for every hub, not only area hubs. Your round-3 support-log entry says `hubs.json` and `hubs.lock` sit in the per-user lym state directory today.
   - Effect: after a deploy, daemons still running the old binary keep writing the old file, and new-binary readers (`hubregistry.Lookup`, the machine walks) read the new one. Hubs that have not restarted drop out of cross-hub creel and `lym machine start`. That breaks the Goal's "with the area off, everything works as it does today".
   - Missing: a migration or first-read step, such as moving or merging the old file, or a stated restart step.

3. **LOW — an area window may lose the attach refusal's message.**
   - Changed text: D6 says "A window opened for an area hub starts no host shell after the attach (D13)", and D13 adds "and no shell after it".
   - Problem: today's Konsole line in `internal/termwindow/termwindow.go` (around line 161) echoes the attach's failure status and ends in `exec bash`, which keeps the window open. Drop the shell and the window closes after its poll loop, possibly before the operator has read the refusal and its way forward (for example D6's `attach-scope` refusal).
   - Missing: keep the window open on a non-zero status without starting a shell, for example by waiting for a key.

4. **LOW — the launcher's failure gives no way forward from inside.**
   - Changed text: D3 says "A launcher that exits fails every later `boot`, and the hub restarts with `lym area stop` and `lym area start`."
   - Problem: `boot` is also called from inside, by reed's spawn and by attach. The text doesn't say that the refusal names this way forward, or that it must be run from a host shell. `lym area start` alone just asks the live supervisor's `boot` again and fails the same way.
   - Missing: say the `boot` refusal names `lym area stop` then `lym area start` from a host shell.

I made no further tool calls; everything above comes from the review, the record as changed and the support log already in context.
