---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: _lyx/plan/02-verify-package-missing.md:79
    summary: The no-finding row go -C .. test ./nope reaches no scan-end branch, since goCommandDirAndPackages returns no package arguments when its -C cannot be placed, and no row covers an unplaceable -C on a lyx gate test line
    fix: In card 2's Tests list, replace the row `go -C .. test ./nope`, where a `-C` that cannot be placed ends the scan, with `go -C .. test ./x && go test ./nope`, where a `-C` that cannot be placed ends the scan before `./nope` is read; and add the no-finding row `lyx gate test -C .. ./x && go test ./nope`, where `gateTestDirAndPackages` reports ok false and the scan ends the same way.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/02-verify-package-missing.md:48
    summary: The Bound paragraph omits the pass the Intent gives every command after an unplaceable -C, and after a cd in a pipeline stage or after ||
    fix: In card 2's Bound paragraph, change "or after an unplaceable `cd` passes" to "or after an unplaceable `cd`, a `cd` in a pipeline stage or after `||`, or a `-C` or `--dir` that cannot be placed passes, since the scan ends there as `verify-nested-module`'s does".
---

The plan carries every decision and constraint of the decision record: D1 and D2 in card 1, D3 in card 2, D4 in card 3, D5 in card 4, D6 and D7 in card 5.
No card's scope reaches past what the record licenses.
`_lyx/loom/rework/` does not exist, so the decision record alone is the answer key.

Focus verdicts:

- Card 2's scan-end rows.
  The pipeline-stage row `true | cd internal && go test ./nope` and the `||` row `false || cd internal && go test ./nope` each report `./nope` or `internal/nope` if the scan went on, whether a continuing scan followed the `cd` or ignored it, since the materialized root holds neither directory.
  The unplaceable-`cd` row `cd $DIR && go test ./nope` reports `./nope` if the scan went on, since `resolveRunDir` rejects `$`.
  The unplaceable `-C` row does not, which is F1.
- Card 2's passes against D3.
  Card 2 states no pass for an argument resolving outside the worktree root: `go test ../outside` is a reporting row, which matches D3's "neither exists on disk under the worktree root".
  Its module-wide-pattern pass is narrowed to a pattern whose stem exists, which matches D3's Decision text, which joins every relative argument after removing `/...`.
  Its gate-line stop at `--` matches `internal/gatecli/cli.go`, where arguments after `--` are go test flags rather than packages.
  The scan end at an unplaceable `-C` is licensed by D3's "cd/-C tracking verify-nested-module uses", since `crossModuleCommands` returns at one, but neither D3's Bound nor the card's lists it, which is F2.
- Card 4's read error after a used re-step.
  `runLoop` clears `restepped` only on an ok child (`internal/shedverbs/loop.go:224`), so after a transient child failure the next top-of-loop read runs with the re-step used.
  The Intent's last read-error bullet states the immediate stop with the read-error way forward, and the subtest "a read error after a used re-step stops without a re-read" reaches it: one child, `status-unreadable: read lock unavailable`, and `loop.restep` naming the child's transient error, which an implementation that ignored the flag would overwrite with the read error.
  Not a defect.

### [LOW] Unplaceable -C row reaches no branch

**Location:** `_lyx/plan/02-verify-package-missing.md:79`

**Issue:** `goCommandDirAndPackages` returns `"", nil, false` when its `-C` cannot be placed (`internal/planparser/verifymodule.go:226-228`), so `go -C .. test ./nope` yields no package argument for `missingPackageArgs` to read.
The row reports nothing whether the scan ends there, skips only that command, or ignores `ok` entirely, and has no later command a continuing scan would read.
The Intent's scan end at "a `-C` that cannot be placed" therefore has no row that would report if the scan went on.
`gateTestDirAndPackages` gets the same ok-false return for an unplaceable `-C`/`--dir`, and no row covers it at all.
Round 1's fix added this row on the premise that it "would report `./nope` if the scan went on"; for this one row the premise does not hold, while the pipeline-stage and `||` rows it added do reach their branches.

### [LOW] Bound omits the scan-end passes

**Location:** `_lyx/plan/02-verify-package-missing.md:48`

**Issue:** The Intent (line 29) ends the scan at a `cd` in a pipeline stage, a `cd` after `||`, an unplaceable `cd` and an unplaceable `-C`, so every later command's arguments pass unread.
The Bound paragraph lists only "after an unplaceable `cd`".
An unplaceable `go -C` changes no shell directory, so `go -C /abs test ./x && go test ./nope` passes `./nope` from a directory the scan could still place.
The rubric's attack-surface rule asks the card that introduces a pass to state it in its bound.
