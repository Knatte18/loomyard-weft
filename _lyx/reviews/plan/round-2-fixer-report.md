# Fixer report — plan round 2

## Changes

- **F1** (fixed as proposed): in `_lyx/plan/02-verify-package-missing.md`'s no-finding rows, `go -C .. test ./nope` became `go -C .. test ./x && go test ./nope`, where a `-C` that cannot be placed ends the scan before `./nope` is read.
  Added the row `lyx gate test -C .. ./x && go test ./nope`, where `gateTestDirAndPackages` reports ok false and the scan ends the same way.
  The premise holds: `goCommandDirAndPackages` returns `"", nil, false` for an unplaceable `-C` (`internal/planparser/verifymodule.go`), so the old row read no package argument whether or not the scan ended.
- **F2** (fixed differently): the Bound paragraph now lists the scan-end passes: after an unplaceable `cd`, a `cd` in a pipeline stage or after `||`, and in or after a command whose `-C` or `--dir` cannot be placed, since the scan ends there as `verify-nested-module`'s does.
  The proposed in-place wording was not used as written.
  Inserted into the existing sentence, its "since the scan ends there" clause would also cover the variable and `bash -c` cases, which pass for a different reason.
  The paragraph now has two sentences, one for static-spelling passes and one for scan-end passes.
  It also names the command carrying the unplaceable `-C` itself, whose own arguments pass too.

## Disputed

None.

## Deferred

None.
