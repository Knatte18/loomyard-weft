```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged each proposal the fixer carried out as written.

**1. MEDIUM: moving the relay state out of reach breaks an in-area `lym slack` reader.**
- Changed text: D12, the bullet "The relay's log, state, lock, outbox, inbox, bridge notices and bridge posts: `<area state dir>/relay/` … unreadable inside" (F39 and F40 as proposed).
- What goes wrong: `internal/slackcli/cli.go:757` calls `slackengine.ReadRelayState(c.geom)` from a `lym slack` verb, and that verb runs in-area.
- With the relay state under an unreadable directory, that verb fails inside an area.
- Only `EnsureRelay`'s reads (`start.go:214`, `:279`) are routed through `slack-relay`. Nothing routes this one.
- The proposal's list is also missing a `status` answer that carries the relay state, and D12 does not say the state is readable inside.
- Evidence: `internal/slackengine/relaystate.go:33`, `internal/slackcli/cli.go:757`.

**2. MEDIUM: the no-symlink rule covers opens only, so host connects into the area root still follow links an agent plants.**
- Changed text: the new D3 bullet "opens nothing under the hub root or the area root through a symlink … `openat2` …" (F39 as proposed).
- What goes wrong: an area hub's creel socket sits directly in its area root, and every area agent can write there (D6, D8 Bound).
- An agent replaces `creel-<sn>-<h>.sock` with a link to a third hub's creel socket.
- Another hub's host-side daemon reads the path from the area registry and calls `connect()` on it. `connect()` resolves symlinks.
- That daemon then delivers mail meant for the area hub into the third hub, under the sender's own identity.
- The rule names "the host creel daemons" but bounds only `openat2` opens, not connects.
- A host process connecting under the area root should connect through an `O_PATH` descriptor opened with `RESOLVE_NO_SYMLINKS`, as D4 does. Without that, the rule does not hold for the daemons it names.
- Evidence: D3's new bullet, D6 "The hub's creel socket sits directly in the area root", D8 Bound "any agent of that area can bind its own listener there".

**3. LOW: granting `~/.gitconfig` as a file breaks area git whenever the host rewrites it.**
- Changed text: D6, "`~/.gitconfig` and `machine.json` are granted as files … unreadable inside until the next `lym area stop` and `lym area start`" (F35's "or state how a replaced file is re-granted" branch).
- Concrete case: `git config --global` replaces `~/.gitconfig` by lock and rename.
- After that, area git loses `user.name` and `user.email`, and commits fail until the operator restarts the area.
- The same goes for any host tool that writes the global config, `gh auth setup-git` for one.
- `machine.json` is safe: lym only reads it outside `lym machine name`.
- No acceptance criterion or D16 note tells the operator about this restart.
- Evidence: `internal/machinename/machinename.go` has only read entry points, and its callers (`beamcli/deps.go:48`, `fabriccli/anchordeps.go:382`) only call `ReadIdentity`.

Calls:
- Read the `bridgeout.go` header and grepped the `ReadRelayState` callers, to learn which side reads the relay files the fix moved. My context had only the writers' paths.
- Grepped `machinename`'s functions and callers, to check whether lym rewrites `machine.json` routinely. My context had only its file and lock names.
