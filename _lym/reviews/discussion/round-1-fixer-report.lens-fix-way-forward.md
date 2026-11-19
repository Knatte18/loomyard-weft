```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked every refusal, wait, next step and criterion the fix introduced.

Ways forward checked:
1. D5: the held-lock wait refuses with the server log named.
2. D5: an `open` during a drain finds the lock free and starts a fresh server.
3. D5: a caller with a strand name stops waiting at the keep-warm cap and returns success.
4. D10: releasing a task in a run status is refused, naming `lym board set-status`.
5. D11: an exiting server answers its waiting long polls with "unchanged", so the page retries and then shows its `lym board open` line.
6. D8: a slug refused for another reason gets the board's own message.
7. D6: a missing task label opens VS Code's task picker filtered by that label.
8. The new criteria in D5, D6, D8, D9 and D10.

Findings:

- **MEDIUM.** D5 second bullet, "A held lock means a server that is starting or serving: `open` waits … until the state answers", together with "On either path, a server that does not answer in the bound is refused with the server log's path named."
  The new held-lock path can refuse on every press with no step that makes progress.
  A server that holds the lock but never answers stays hung: it is stuck, not crashed, so its lock stays held.
  Before the fix, the only non-answering case was a server `open` had just started, and its death freed the lock.
  Now a hung server makes every `open` refuse.
  The only way forward is the log path, and D11 says the server has no stop verb ("without an operator stop verb").
  The idle exit cannot be counted on either, because a server that answers no request also runs no timer the record ties to its requests.
  The refusal should also name how to end the stuck server, such as its pid from the state file or the `runproc` record, or the record should state the bound.
- **LOW.** The D5 criterion "with a strand name set and an opener that does not exit, `open` returns success at the keep-warm cap for that role."
  `shuttleengines.KeepWarm` (`internal/shuttleengines/keepwarm.go:31`) returns `shuttleengine.DefaultAwaitCap` as its fallback, and the record names no seam to shorten the cap.
  As written, the test must either wait out the real cap or find its role's engine answering zero (`Cap` zero means "keeps nothing warm").
  The record should say the cap is injected in the test, so the criterion can run quickly.
  The zero-cap case does still make progress: `open` returns at once.

The other items lead somewhere reachable.
D10's release refusal names a verb that acts on running tasks; the old D10 bound said `set-status` acts on a running task.
A drain hands off to a fresh server.
The page's `lym board open` line reaches the new server.

Tool calls:
- grep of `internal/shuttleengines` and `internal/boardcli`: my context did not show whether the keep-warm resolver exists or where `set-status` lives.
- sed of `internal/shuttleengines/keepwarm.go:1-60`: my context lacked the resolver's fallback and the meaning of a zero cap.
