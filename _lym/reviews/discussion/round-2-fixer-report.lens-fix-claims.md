```yaml
lens: fix-claims
outcome: reported
```

Lens: fix-claims. I checked that each claim the fix added or rewrote holds against the code at HEAD.

**MEDIUM: the D7 ancestor walk can miss an area hub reached by a symlinked path.**
- Changed text: D7 says the walk "refuses when any of them has a hub key, derived from its path as `reedengine.ServerName` derives it, with an area file".
- Problem: the walk runs over the *canonical* cwd's ancestors. `ServerName` hashes `filepath.Abs(hubPath)`, which does not resolve symlinks (`internal/reedengine/server.go:47-52`, `cleanAbsHubPath` at `:304-313`). D2 names the area file after "reed's server name for the hub". If that name was derived from a non-canonical hub path, the walk derives a different key from the canonical ancestor, finds no area file, and does not refuse. The bound holds only if D2 states that the hub key is derived from the canonical hub path.

**MEDIUM: "only the supervisor makes a scope under the prefix" is false as stated.**
- Changed text: the D3 inside-test bullet, "No area process can leave its scope, … and only the supervisor makes a scope under the prefix".
- Problem: any host process that can reach the session bus can start a scope with any unit name, including `lym-area-&lt;hub key&gt;-…`. An agent of a hub without an area is one example. The record's own premise is that `systemd-run --user` can be reached outside an area (Goal; D5 rationale). What holds is narrower: no *area* process can make one, since the bus is refused inside (D5). The result is that a host process with any seccomp filter inside such a scope is judged inside. It is then refused, not let through, and `lym area stop`'s list-by-name ends it. The sentence should say "no area process can make a scope under the prefix" rather than claim the supervisor is the only maker.

These claims hold:
- **Round 2 support log, `ServerName`:** it derives the hub key from the hub path alone (`server.go:47-52`).
- **Round 2 support log, attach line:** the worktree path enters the attach line only shell-quoted (`termwindow.go:152`). Konsole gets it as its own `--workdir` argv element (`termwindow.go:96`), so the title was the only open Konsole argument.
- **D8, the receive path's hub reads:** an area hub's identity is read only through `s.hubs.Lookup`, `s.hubs.Geometry` and `s.hubs.RepositoryName`, plus the receiver's own `TrustedHubs`. The new bullet names all three of the first group.
- **D12/D16, the per-orch channel:** it is public (`api.go:298` sends `is_private=false`). It is named from the board project (`keeper.go:151-155`) and the operator is invited (`keeper.go:169`).
- **D12/D16, cross-machine mail:** it creates bridge channels (`bridgechannel.go:54,91`) and invites the peer's bot (`bridgeout.go:287`).

Tool call: one Bash grep, which read `cleanAbsHubPath`, the `s.hubs.*` call set in creelengine, and `CreateChannel`'s `is_private`. My context did not hold the path canonicalization, the full set of hub reads on the receive path, or the channel privacy flag.
