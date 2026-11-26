```yaml
lens: fix-claims
outcome: reported
```

Fork lens fix-claims: I checked the claims the fix added or rewrote against the code at HEAD.

**LOW.** The changed text is D6's new attach bullet: "and `boot` sets the server's `update-environment` empty, so attaching copies no host variable into a session."
- **Summary:** this claims more than the mechanism holds. The client's allow-listed variables (`TERM`, `LANG`, `LC_*`, `TZ`) are host variables too. And an agent can set `update-environment` again through the hub's own tmux socket, because D8's Bound already says any agent can drive the server.
- **Evidence:** D8 Bound: "any agent can already drive the orch's pane through the hub's own tmux socket". What actually holds is the client's allow-listed environment, so "no host secret" is the claim that stands, not "no host variable".

**LOW.** The changed text is D6's new bullet: "a reader of the machine record or the area registry takes its shared lock on a descriptor opened read-only."
- **Summary:** a read-only open cannot create the lock file. A missing `hubs.lock` or registry lock therefore fails the read inside, and the decision does not say that the host writer creates the lock first.
- **Evidence:** `internal/lock/lock.go:35` takes its locks through `flock.New(lockPath)`, which opens the file with create and read-write by default. No changed text names who creates the lock file before an area reads it.

Every other claim I checked holds:
- `--hub-path` exists in `internal/slackcli/cli.go`, `internal/creelcli/cli.go` and `internal/reedcli/watchdog.go`.
- `lym slack away` exists (`internal/slackcli/cli.go:599`).
- The relay is what writes the bridge map (`internal/slackengine/bridgeout.go:1`, "the relay's sending of machine mail").
- The presence and peers files sit in the credential directory, and so does the socket lock (`internal/hubgeom/slackgeom.go:26-29`).
- Go takes `clone3` only on the `UseCgroupFD`, `CLONE_NEWTIME` or `forceClone3` condition (`syscall/exec_linux.go:315`).
- The exit hook claims hold: `cmd/lym/exitsweep.go:20` and `logger.NotifyExit`.
- D12 now says in-area presence reads go only through the verb. dobby reads presence directly today (`internal/beamcli/dobbydeps.go:549`), which is consistent as a design change and not a false claim.

Tool calls:
1. Grepped for the `--hub-path` flag, the `away` verb, the Slack map writers and the presence readers, because my context did not show which code holds them.
2. Read the header of `internal/slackengine/bridgeout.go` to learn which process writes the bridge map, which my context lacked.
