# Fabric's Windows path behaviour verified on a Windows host

`fabric-windows` · Note · enhancement, fabric

From `manifest/roadmap.md` Someday, kept by the operator on 2026-10-03: not a priority now, taken only if Windows becomes relevant.
Fabric's Windows path behaviour (directory junctions through `internal/fslink`) is unverified after six hardening rounds; closing it needs a Windows host, not further design.
The design file this note once cited, `manifest/designs/fabric-windows-verification.md`, no longer exists in the repository.
Check on a Windows host that a directory junction's own modification time stays fixed when files are created, changed or removed under its target.
The root `.scratch` link fabric wires into the weft `.lyx` relies on it: source-scan tests hash the module root's `.scratch` entry into their `go test` cache key, and a junction whose own time moved would defeat the cache again.
