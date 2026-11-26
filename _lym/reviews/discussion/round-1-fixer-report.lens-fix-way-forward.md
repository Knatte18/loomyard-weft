```yaml
lens: fix-way-forward
outcome: reported
```

Task: check every way forward (refusal, next step, row, test) the fix introduced, using the fix-way-forward lens.

1. **MEDIUM**, D6, the attach bullet ("skips `EnsureSession`, `Status` and the watchdog spawn; with no server answering it asks the supervisor's `boot` verb"), together with D13 `window`.
   - **Summary:** host-side attach can no longer reach a cold worktree of a running area hub.
   - **Evidence:** `EnsureSession` (`internal/reedengine/lifecycle.go:796-807`, `ensureSessionLocked` at 784) boots this worktree's session whenever nothing usable is there to attach to, not only when no server runs. D3 says `boot` "takes no argument" and "does nothing otherwise" once a server runs.
   - **Effect:** host-side attach and D13's `window` on a worktree with no session in a live server have no verb that creates that session. The change names no way forward, such as running `lym reed` from a shell window of the server, or a `boot` that takes the worktree.

2. **MEDIUM**, D7 ("skips each area hub with a notice naming the same way forward", that is, "run it from a shell window of the hub's server").
   - **Summary:** after a reboot, `lym machine start` points at a server window that does not exist yet.
   - **Evidence:** in that case no server runs for the area hub, so there is no shell window to run from.
   - **Effect:** the notice cannot be acted on as written. It needs to name `lym area start` (or the attach that boots through `boot`) first, then the in-area step that pulls the board and starts worktrees. As written, the area hub's machine-start work has no named route.

3. **MEDIUM**, D10 ("the area's Claude config directory installs its own plugins", "Claude Code's plugins installed, once inside the area").
   - **Summary:** a plugin whose marketplace is a local directory outside the hub cannot be installed from inside the area.
   - **Evidence:** `~/.claude/plugins/known_marketplaces.json` has the `loomyard` marketplace as `"source": "directory", "path": "/home/knatte/Code/loomyard-LYMHUB/loomyard"`. D6's read list does not include that path for any other hub.
   - **Effect:** for any area hub other than this one, installing `ly` fails. Nothing says how a later deploy's plugin update reaches the area's copy either.

4. **LOW**, D3 (the peer-check bullet lists `lym area stop` among "every host-side caller of a supervisor verb … refuses otherwise") against D3's stop ("whether the supervisor is alive or dead") and the dead-supervisor acceptance criterion.
   - **Summary:** it is unclear whether `stop` contacts the supervisor at all.
   - **Effect:** with a dead supervisor (or a pid reused since), a `stop` that runs the peer check refuses, which blocks the recovery D4 names ("restarts with `lym area stop` and `lym area start`"). The text should say that `stop` uses no verb, or that it goes on to the scope stop when the peer check fails.

Further tool calls:
- Bash: grep of `EnsureSession` in `internal/reedengine/lifecycle.go`. My context had the attach call order but not what `EnsureSession` boots.
- Bash: `ensureSessionLocked` plus the head of `~/.claude/plugins/known_marketplaces.json`. My context lacked `EnsureSession`'s cold-worktree behaviour and the plugin marketplace sources.
