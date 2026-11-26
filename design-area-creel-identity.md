# creel across an area: kernel-attached sender identity, the area registry and the hub directory

`area-creel-identity` · Task · feature, creel, cwd · draft

Decision record: [Sketch](decision-area-creel-identity.md)

Filed from the Sketch `sandbox-closed-area-build`.

Takes: the Sketch's decision record, D8 whole and the pieces of D3 and D6 that D8 rests on; peerPID (internal/creelengine/peer_linux.go), daemon.resolve and resolveSender, serveHubCall (receive.go), the outbound worker's spawn closure (outbound.go attempt), hubregistry's Entry, Lookup, Live and RepositoryName, creelcli's hubdirectory and its one machine-record write
Delivers: every creel daemon sets SO_PASSCRED and SO_PASSPIDFD and takes the sender from SCM_CREDENTIALS and SCM_PIDFD with the pidfd liveness check; D3's inside test (barrier in /proc/<pid>/status or attr/current, plus a cgroup naming a scope of the hub's area by the whole-name forms lym-area-<hub key>.scope and lym-area-<hub key>-<pane|run|attach>-<pid>.scope), with which a daemon serves only deliver and confirm to a peer inside another hub's area or a peer gone or unreadable; the machine record moved to the hub directory <per-user lym state dir>/hubs/ with its lock, read under a shared lock on a read-only descriptor, with the old hubs.json read for entries the new record lacks; each daemon writing its shortname, creel socket path, repository name and the area-aware creel level into its entry, bar an in-area daemon; the area registry area-hubs.json and its lock in the hub directory, its reader, and a writer the core part's lym area start calls; hubregistry.Lookup, the confirm's geometry and RepositoryName reading an area hub's registry entry and, inside an area, every other hub's host-written record, never a Bolt shortname file or a prime's .git/config; every registry reader and writer dropping an entry whose area file ~/.config/lym/area-<hub key>.yaml is gone (Added after Discussion); the outbound worker starting no other hub's daemon inside an area, and a hub without an area starting no area hub's daemon; PATTERN-agent-addressing's Across hubs background gaining the area-registry repository rule
Exports: areaengine.HubKey (reedengine.ServerName of the canonical hub path), areaengine.FilePath(hubKey), areaengine.ScopeName and the scope-name parser, areaengine.Inside(pid, hubKey, barrier); hubregistry.AreaEntry {hub_path, shortname, creel_socket, repository}, hubregistry.ReadAreaRegistry and hubregistry.WriteAreaEntry, the hub-directory path helper; the area-aware creel level constant D5 admits

The part changes these packages:

- internal/creelengine (about 450 changed lines)
- internal/hubregistry (about 350 changed lines)
- internal/creelcli (about 150 changed lines)
- internal/areaengine (about 250 changed lines)
- internal/lymdirs (about 20 changed lines)
- internal/machinecli (about 30 changed lines)
- internal/slackcli (about 20 changed lines)
