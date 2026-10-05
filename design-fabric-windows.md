# Fabric's Windows path behaviour verified on a Windows host

`fabric-windows` · Note · enhancement, fabric

From `manifest/roadmap.md` Someday, kept by the operator on 2026-10-03: not a priority now, taken only if Windows becomes relevant.
Fabric's Windows path behaviour (directory junctions through `internal/fslink`) is unverified after six hardening rounds; closing it needs a Windows host, not further design.
See `manifest/designs/fabric-windows-verification.md`.