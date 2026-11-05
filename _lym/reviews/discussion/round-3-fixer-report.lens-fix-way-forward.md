```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: checked whether each refusal, next step, row and test the fix added can be carried out and leads to progress.

**Ways forward the fix introduced:**
1. D2: once the worker has started the receiving daemon three times, it only reconnects. A new post or a sending daemon restart resets the count. The message returns undelivered at the hour.
2. D2: the return notice's reason is either a creel code or the receiver's quoted text. The notice still names `lym creel log --id <id>`.
3. D3: the fifth drop notice says later drops are only journaled and names `lym creel log`. The first drop after the hour sends a counted notice.
4. D3: an identical message from a hub whose ids restarted is refused as an id clash. The existing id-clash refusal names its way forward.
5. D7 and D9: sessions launched before the deploy restart before another hub's orch mails them. The plan's deploy note says the same.
6. New criterion rows in D2 and D3: the start cap across the hour, the quoted reason, the five-notice flood, and the identical-message id clash.

**Findings:**

- **MEDIUM** — D7 bound and D9 setup: "every session of the machine's hubs launched before the deploy restarts before an orch of another hub mails it (D7, D9)."
  - The step cannot be done as D7 states it.
  - Once the deploy restarts a daemon, that daemon takes cross-hub mail from any orch whose hub is in D1's record. Nothing in the design holds delivery back until the restarts are done.
  - D9 orders the restart only before "the trial's first cross-hub post". An orch in `lyt` or `cs`, or any other hub, can post on its own initiative in the window between the deploy and the restarts.
  - D7's bound promises more than the rollout step can give. Either state that the window between deploy and restart stays exposed, or name something that holds cross-hub delivery until the restarts are done.
  - Evidence: D2's outbound worker delivers every pending message as soon as the receiving daemon answers, and D3 step 6 stores it. No step checks the receiving session's launch time or a rollout flag.

- **LOW** — D3 drop-notice bound: the fifth notice "names `lym creel log` as the way to them".
  - The way forward works but is not focused. `lym creel log` has no filter by sending hub; its filters are `--id`, `--from`, `--to` and `--since` (`internal/creelcli/cli.go:732-733`).
  - Bare `lym creel log` prints the whole journal, so the orch has to search all of it for the drops that came without a notice.
  - Naming `lym creel log --since 1h` would point straight at them.

Tool calls beyond context:
- grep of `internal/creelcli/log*.go` for `lym creel log`'s flags: no such file existed, so I listed the directory instead. My context did not show where the log verb's flags are defined.
- grep of `internal/creelcli/cli.go` for the log verb's flags: to check whether the `lym creel log` way forward can filter by hub. My context did not hold the verb's help text.
