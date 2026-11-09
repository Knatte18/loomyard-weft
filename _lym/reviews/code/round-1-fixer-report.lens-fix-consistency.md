```yaml
lens: fix-consistency
outcome: reported
```

Task: lens fix-consistency, checking the change set for stale twins and cross-references.

**Findings**

1. LOW. `internal/dobbyengine/loop.go:124-125`, the `Deps.StartTestMap` comment: "The process is registered for the hub, so TestMapRunning reads it live". The F1 change makes a failed registration return nil with the child left unregistered. A nil return then no longer means `TestMapRunning` will read the refresh live. The `Deps.StartTestMap` text, `dobbyengine/doc.go` and `beamcli/doc.go` all state the new rule, but this seam contract still states the old guarantee unconditionally. Fix: say it is registered "unless registration fails, in which case it ends on its own".

2. LOW. `docs/overview.md:803`, "Stop takes a beam's orch and dobby down." This is the stop twin of the F3 change. The package docs (`beamcli/doc.go:320`, `orchcli/doc.go:43`, `orchcli/stop.go:37`) now say the prime's stop also ends the test-map refresh, whether or not the dobby is live. Line 807 of the overview describes the refresh, but the stop line does not mention it. The F3 fix did not carry over to the overview.

**Checked, no findings**
- `endDobbyRoles` has no remaining reference in code or docs. The `runproc` import was removed from `stop.go`, and `cli.go` still uses it.
- "ends it again" is gone from every doc.
- `runproc.EndProcess` has no remaining caller in `internal/beamcli`.
- `fabriccli/doc.go` (the F6 change) agrees with `dobbyengine/doc.go` and with `fabriccli/anchor.go:468`, which spares only the push and sync roles.
- The `tick` comment, `doc.go:108` and `doc.go` (the `moving` text) agree on the test-map skip.

**Further tool calls**
1. A single Bash call (grep for `endDobbyRoles`, `ends it again`, `unregistered`, `EndProcess` and the test-map/stop mentions, plus `docs/overview.md` lines 803-812). My context did not hold `docs/overview.md`, or the repo-wide set of stale references.
