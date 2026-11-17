Lens fix-consistency.

1. LOW: `internal/beamcli/doc.go`, the `stop --all` section, was stale: the detached process starts at the prime's anchor, step 2 did not say a task pair is now classified by a readable origin record, and step 3 did not say a failed dobby step no longer skips the orch process end.
2. LOW: `docs/overview.md` line 850, the hibernate bullet, lists only `request`, `cancel` and `status` while the line 385 tree line now names the hub pause; not a contradiction.

Tool calls: one grep and sed over `beamcli`, `orchcli` and `docs` for twin wording, since `beamcli/doc.go` was not in the lens's context.
