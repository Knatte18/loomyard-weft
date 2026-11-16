```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens on the merge 4153c5ef and commit c1a136c7. For each change, I checked what calls, reads, asserts or documents it.

**Affected set, per change**

- **`resumeOrch(sessionID)` became `resumeOrch(sessionID, remoteControlName)`** (`internal/orchcli/start.go:761`). Covered.
  - The only caller is `launchOrch` at `start.go:734`, which passes `remoteControlName`. No test calls `resumeOrch` directly.
  - The retry after `endOwnHolder` reuses the same `spec`, so it also carries remote control (`start.go:776`).
  - The package doc agrees. `internal/orchcli/doc.go:276` says "A launch, fresh or resumed, names the remote session", and `doc.go:72-73` still describes the holder end and second resume correctly.
  - `orchSpec` already takes three arguments (`start.go:214`).
  - `go build` and `go vet` of `./internal/orchcli` pass, and vet compiles the test files too.
- **The `endTestMap` field and its wiring were dropped** (`internal/orchcli/cli.go`). Covered.
  - Main removed its one consumer, in `beamDown` (`stop.go`), along with the doc lines in `BeamDown` and `beamDown`.
  - grep finds no `endTestMap` left in `internal` or `cmd`.
- **`RoleTestMap` was removed from `subjectPassSpared`, and the "coverage-map pass" wording from the `EndSubject` doc** (`internal/runproc/end.go:88,100`). Covered.
  - Main deleted `RoleTestMap`, and grep finds no reference left.
  - The `EndSubject` paragraph in `internal/runproc/doc.go:61-62` matches.
  - `internal/pairteardown/doc.go:284-290` names no spared role.
  - `pattern/PATTERN-child-processes.md:34-58` names no test-map role.
  - grep finds no "coverage-map" left in `internal`, `pattern` or `PATTERN.md`, apart from `internal/gatecli` and `internal/testcover`, which are the map tool itself.
- **The hub roles list in `internal/runproc/doc.go:9-10` now follows main, and the beam's `RoleOrch` line is kept.** Covered. It matches the remaining roles: no `RoleTestMap` exists.
- **The `RoleTestMap` record was removed from the fixture of `TestEndSubject_EndsTheRunsThenTheSubjectAndItsDobbyLast`** (`internal/runproc/end_test.go:479-485`). Not covered: the subtest name was left stale (finding 1).
- **c1a136c7 rewrote the `resumeOrch` doc comment** (`start.go:756-757`). Covered. It touches only the comment, and no reader depends on its text.
- **Rule checks.** Covered.
  - Declared PATTERN changes: the directive declares none.
  - `Pattern-Departure` trailers: neither commit in the range carries one, and no difference needs one.
  - Stencils: the diff touches none.

**Findings**

1. **LOW, consistency.** `internal/runproc/end_test.go:524`: the subtest name `TheHubOwnerSubjectLeavesTheHubWidePushesTestMapSlackRelayAndTeardownAlone` still names "TestMap". The merge removed the `RoleTestMap` record from the `HubOwner` fixture (`end_test.go:479-485`; the remerge-diff of 4153c5ef drops `{PID: 81, ..., Role: RoleTestMap}`), so the name now claims a behavior that no row tests and no role exists for.
   - Fix: drop "TestMap" from the name, for example `TheHubOwnerSubjectLeavesTheHubWidePushesSlackRelayAndTeardownAlone`. The merge's own removal made the name stale, so the merge needs this change.

**Tool calls**

1. grep across `internal`, `pattern`, `PATTERN.md` and `docs` for "coverage-map", "TestMap" and `EndSubject` descriptions. My context held no search of the docs or pattern files outside orchcli and runproc.
2. Read `internal/pairteardown/doc.go:280-295` and `internal/orchcli/doc.go:105-125`, plus a grep of `internal/orchcli/doc.go` for resume and remote-control lines. I needed the doc text that describes `EndSubject` and the resume and remote-control launch, which my context lacked.
