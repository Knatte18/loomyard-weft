```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the merge and the follow-up commit to the board entry's goal points.

**Per change**
- **C1, merge e6e36704, conflict in `internal/machinename/machinename.go`.** Traces to G1. The resolution keeps the beam's `idBytes` and main's exported `MaxNameLength`. It drops the beam's unexported `maxNameLength`, which nothing uses any more: the grep found no reference left, and `CheckName` uses `MaxNameLength` at both lines 129 and 135 of HEAD.
- **C2, commit 26b29bb8, restores `Names` in `internal/machinename/machinename.go:120-126`.** Traces to G2. Main brought in two callers of `machinename.Names`, at `internal/creelcli/slackbridge.go:36` and `internal/slackcli/wiring.go:381`, and the beam's 6005496f had removed the function. The restored function and its doc match main's `machinename.go:79-85` and the removed body in 6005496f word for word. It is a forced fix, inside G3.

**Per decision**
- **G1, run `lym fabric merge-in main` and settle conflicts:** built by C1.
- **G2, make the gate pass and fix what the merge broke:** built by C2. `go vet` passes on machinename, creelcli, slackcli, creelengine and slackengine.
- **G3, no change the merge does not need:** both C1 and C2 hold to it. Neither adds a refactor, a feature or an older-fault fix.
- **G4, an up-to-date merge changes nothing:** does not apply, because the merge brought in commits.
- **G5, Code-Review reviews the merge alone:** this is a process directive, not a code change, so there is nothing to build.
- **Added after Discussion entries:** the record has none.

**Findings:** none.

**Tool calls:** none. The inherited context already held the diff, the record and the caller greps.
