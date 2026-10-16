---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/02-verify-package-missing.md:48
    summary: Card 2's Bound paragraph omits four exemptions its Intent grants, one of them D3's own Move bound, and the spec row copies that paragraph as the bound above
    fix: In _lyx/plan/02-verify-package-missing.md, extend the Bound paragraph so it also lists every exemption the Intent grants. These are a Verify on or after a card with a Move group, and the overview chain when any card has one; every command after a cd in a pipeline stage or after ||; a go or lyx gate test -C/--dir that cannot be placed, and every command after it; and a lyx gate test field after --. The loom-plan-spec item 42 that the Docs paragraph builds from the bound above then carries all of them.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/02-verify-package-missing.md:79
    summary: The unplaceable -C row cannot tell ending the scan from skipping one command, and gateTestDirAndPackages' ok-false branch has no row at all
    fix: In card 2's Tests list, replace the row `go -C .. test ./nope` with `go -C .. test ./x && go test ./nope`, which reports nothing only if the scan ends at the -C as the walk paragraph says. Add the no-report row `lyx gate test -C .. ./x && go test ./nope`, which reaches gateTestDirAndPackages' ok-false branch and pins that it ends the scan too.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/04-loop-status-stop.md:69
    summary: The read-error-after-a-used-re-step subtest does not assert the read-error way forward that card 4 says that stop carries
    fix: 'In card 4''s subtest a read error after a used re-step stops without a re-read, add the assertion that the error contains `way forward: re-run lyx shed step run-1 --until-stop`. Without it, a used-re-step branch that leaves statusErr nil would get the status-missing reseed clause from statusStopWayForward and still pass.'
---

### [LOW] Card 2's Bound paragraph is narrower than what the card lets through

**Location:** `_lyx/plan/02-verify-package-missing.md:48`

**Issue:** The Bound paragraph lists a variable, `bash -c`, an unplaceable `cd`, an import path, a module-wide pattern whose stem exists, a `.go` file argument and an earlier card's delete.
The Intent also exempts more.
A card Verify on or after a Move card, and the overview chain when any card has one, are not checked; that is D3's own second bound item, and the Bound paragraph does not repeat it.
Every command after a `cd` in a pipeline stage or after `||` passes.
A `-C` that cannot be placed ends the scan, for `go` and for `lyx gate test` through `gateTestDirAndPackages`' ok false, so its own command and every later command pass.
`lyx gate test` fields after `--` pass.
Each one is licensed: D3's decision body adopts `verify-nested-module`'s chain splitting and `cd`/`-C` tracking, and `--` is where `lyx gate test`'s own package list ends (`internal/gatecli/cli.go`, `ArgsLenAtDash`).
The Bound paragraph is still the card's one statement of what the guard lets through, and the Docs paragraph builds `loom-plan-spec.md`'s item 42 from "the bound above".
So the spec row would leave out D3's Move bound and the scan endings at `-C`.
The `-C` scan ending matters most, because a `go -C` does not move the shell: `go -C .. test ./x && go test ./nope` lets `./nope` through, and D3's bound names only an unplaceable `cd`.

### [LOW] The unplaceable `-C` rows do not pin the scan ending

**Location:** `_lyx/plan/02-verify-package-missing.md:79`

**Issue:** The walk paragraph (line 29) ends the scan at a `-C` that cannot be placed.
The `cd` rows each put `go test ./nope` after the ending `cd`, so continuing the scan would report it:
- `cd $DIR` fails `resolveRunDir` on `$`, and `./nope` at the root is missing.
- After `true | cd internal` and `false || cd internal`, `internal/nope` is missing.
The `-C` row `go -C .. test ./nope` is one command.
An implementation that skipped only that command, `continue` instead of `return`, passes it unchanged, so the row cannot tell ending the scan from skipping one command.
`gateTestDirAndPackages`' own ok-false branch (`-C`, `-C=`, `--dir`, `--dir=` with an unplaceable value) has no row, so no test reaches it.

### [LOW] The used-re-step read-error stop's way forward is not asserted

**Location:** `_lyx/plan/04-loop-status-stop.md:69`

**Issue:** Line 27 says a read error after the re-step is used "stops at once with `status-unreadable: <error>` and the read-error way forward".
The subtest for that case asserts the cause and `loop.restep`, but not the way forward.
`statusStopWayForward` picks its clause from `res.statusErr`, and a nil error selects the absent-file reseed clause.
So a branch that set the cause but left `statusErr` unset would name `lyx loom start` or `lyx shed seed` for a readable, running file, and the subtest would still pass.
The other branch, the re-read failing in "a read error that fails its re-read stops", does assert its way forward.

## Focus verdicts

- Card 2's exemptions against D3's bound: each exemption is licensed by D3, through its bound list or through its adoption of `verify-nested-module`'s `cd`/`-C` tracking. The narrowing to "a module-wide pattern whose stem exists" follows D3's own trailing-`/...` rule. The card's Bound paragraph does not carry all of them (F1).
- Card 2's chain-walk endings: the pipeline `cd`, `||` `cd` and unplaceable-`cd` rows each put a command after the ending that would report if the scan went on. The `-C` row does not, and `lyx gate test`'s unplaceable `-C` has no row (F2).
- Card 4's status read error: the card states both outcomes. With the re-step unused, it re-reads and steps on or stops; with the re-step used, it stops at once with no re-read. I checked that against `runLoop`'s `restepped` flag, which a successful child resets. Named subtests reach each outcome ("fails its re-read", "clears on the re-read", "after a used re-step"). Only the used case leaves its way forward unasserted (F3).
