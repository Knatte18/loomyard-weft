---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/fabricengine/commitweftat_test.go
    summary: the seed-failure branch of commitWeftAt, which fails the commit before staging, has no test
    fix: Add one row or test to commitweftat_test.go that makes seedWeftArtifactExcludes fail, for example by making the repo's info/exclude path a directory so the read fails, and asserts that commitWeftAt returns a non-nil error, committed=false, an empty sha, and leaves a dirty file unstaged.
    origin: lens:code-tests
---

### [LOW] Seed-failure path of commitWeftAt is untested

**Location:** internal/fabricengine/commitweftat_test.go
**Issue:** D2 states that a seed failure fails the commit, and the diff adds that branch at `internal/fabricengine/recordsgit.go:376-378`.
No test induces a seed failure: `commitweftat_test.go` and `bolt_integration_test.go` cover only the success, no-op and `SkipGit` paths.
The rubric's test economy rule names a behavior the diff adds with no coverage as a finding.

## Rejected

- Lens code-pattern-fit, LOW consistency, that the scenario step touches process-global logger state without naming it, and that the unconditional `SetDurableSinkDir("")` reset could clobber another test's override: no other test in `internal/fabriccli` sets or reads the sink (checked by grep, only `hubmutation_integration_test.go` references it), the steps are serial under one parent, and the reset is the cleanup the record's D1 asks for.
  The sink now points at the test hub's own temp logs, so a parallel test emitting records in the window writes only into a directory removed with the hub.
- Lens code-correctness, unverified remark that `resolveSinkDirLocked` may not give the override precedence: read at `internal/logger/sink.go:172`, `sinkDirOverride` wins, so the `TraceDir` assertion checks the redirect.
- Lens code-correctness, unverified remark that `newBoltRepo` may commit a tracked file that breaks the `LsFiles` equality: read at `bolt_integration_test.go`, the fixture only runs `git init` and `remote add`, so `ls-files` after the commit is exactly `bolt-note.md`.
- Handler check of the seed error being returned bare from `commitWeftAt`: `seedWeftArtifactExcludes` already wraps its error with `fabricengine: seed weft artifact excludes in %s`, and the sibling `StageAllAndCommit` result is returned bare too, so there is no inconsistency.
- Handler check of scope against D1 and D2: the diff touches only the six files the record's two decisions require, with no logger, `boardengine` or bench-file change, and the bench link breaks stay with main as the `Added after Discussion` entry says.
- Handler check of docs: `doc.go`, `bolt.go` and `recordsgit.go` agree with what `commitWeftAt` does, and no remaining comment says Bolt's commit seeds nothing.
