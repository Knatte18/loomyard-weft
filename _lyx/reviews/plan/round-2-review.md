---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/02-verify-package-missing.md:26
    summary: Card 2 prescribes guard branches with no test row that reaches them - gateTestDirAndPackages' unplaceable -C/--dir, the copied walk's bare or multi-argument cd, and the -C row cannot tell ending the scan from skipping one command
    fix: In card 2's Tests, under Rows that report nothing, add `lyx gate test -C .. ./nope && go test ./nope2` (an unplaceable gate -C ends the scan, so neither argument is reported), `lyx gate test --dir $X ./nope` (an unplaceable --dir passes), and `cd internal extra && go test ./nope` (a cd that is not exactly `cd <dir>` ends the scan); change the row `go -C .. test ./nope` to `go -C .. test ./nope && go test ./nope2` so it pins that the scan ends rather than that one command is skipped; extend the closing sentence at line 82 to name these rows as reaching their own branches.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/02-verify-package-missing.md:48
    summary: Cards 2 and 3 strip only a trailing /..., so a valid Go pattern with ... inside a segment, such as ./internal/plan..., is reported by the plan check and refused by lyx gate test, against D3's bound that a module-wide pattern passes and D4's bound that patterns pass to go test unchanged
    fix: In card 2's missingPackageArgs paragraph (line 31) and card 3's missingPackageDirs paragraph (_lyx/plan/03-gate-test-package-dirs.md:16), state that an argument holding `...` anywhere other than as a trailing `/...` is not read and passes; in card 2's Rows that report nothing add `go test ./internal/x...` and `lyx gate test ./nope...`; in card 3's `TestGateTest_Scenario` outside-every-hub row add `./pkg...` to the arguments the stand-in must receive; reword both Bound sentences (card 2 line 48, card 3 line 20) to name the pattern forms that pass.
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/02-verify-package-missing.md:29
    summary: Card 2 copies crossModuleCommands' chain walk into a second loop, so cd tracking and its scan-end rules live in two places that must stay in step, while the walk now has two real call sites
    fix: In card 2, add under Create `plan:internal/planparser#chainCommands` -> `func chainCommands(chain string) []chainCommand` and `plan:internal/planparser#chainCommand` -> `type chainCommand struct`, both in `internal/planparser/verifymodule.go`; chainCommand holds a command's fields with leading env assignments removed and the run directory it runs in, and chainCommands follows a literal `cd <dir>` and ends at a cd in a pipeline stage, after `||`, not of the form `cd <dir>` or unplaceable; rewrite the Intent so crossModuleCommands and missingPackageArgs both range over chainCommands and each ends its own scan at an unplaceable -C. Checked with `lyx loom validate-plan --overlay`, no new finding.
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/04-loop-status-stop.md:39
    summary: Card 4 does not state or test that a read error's re-step, once its re-read clears, leaves no re-step for the next child's transient failure, a change to how the existing transient re-step behaves
    fix: In card 4's read-error bullet (line 39), add that a re-read that succeeds leaves the re-step used until a child succeeds, so a transient failure of the child that follows stops with LoopStopError and writes the status failed; add a row to `TestLoop_TransientRestepOnce` where ReadStatus fails its first call, the next read delegates to readStatusAt, and the one child returns a transient failure through `failing`, asserting LoopStopError, the status failed and `loop.restep` naming the read error.
  - id: F5
    severity: NIT
    class: consistency
    location: _lyx/plan/04-loop-status-stop.md:59
    summary: Card 4 places its read-error re-step rows under TestLoop_InterruptedChildFillsTheErrorForm, though the clears-on-re-read row ends halted and the re-step-once behavior has its own table, TestLoop_TransientRestepOnce
    fix: In card 4, move the rows a read error that clears on the re-read steps on and a read error after a used re-step stops without a re-read into `TestLoop_TransientRestepOnce` as table rows, add `internal/shedverbs#TestLoop_TransientRestepOnce` under Edit, and keep the three stop rows under TestLoop_InterruptedChildFillsTheErrorForm.
---

### [MEDIUM] Card 2's guard branches are not all reached by a named test input

**Location:** `_lyx/plan/02-verify-package-missing.md:26`

**Issue:** The focus asks whether every guard branch of `missingPackageArgs`' walk has a named input in `TestValidate_VerifyPackageMissing`.
The four branches the card lists reach their own code for a `go` command: `cd $DIR` (unplaceable, via `resolveRunDir`'s `$` rejection), `true | cd internal` (separator `|`), `false || cd internal` (separator `||`) and `go -C .. test` (`goCommandDirAndPackages` returns ok false for `..`).
Three gaps remain.
`gateTestDirAndPackages` has its own ok-false branch for an unplaceable `-C`, `-C=`, `--dir` or `--dir=`, and every `lyx gate test -C`/`--dir` row in the card uses the placeable `m`, so that branch has no input.
The walk is prescribed as `crossModuleCommands`' copy, which also ends the scan at a `cd` whose field count is not two (`internal/planparser/verifymodule.go`, `len(fields) != 2`), and neither the new rows nor `TestValidate_VerifyNestedModule` reach it.
The `go -C .. test ./nope` row would report nothing whether the walk ends the scan or skips that one command, so the card's claim at line 82 that it "would report `./nope` if the scan went on" does not hold for it; only a following command distinguishes the two.

### [MEDIUM] A `...` pattern not after a slash is reported and refused

**Location:** `_lyx/plan/02-verify-package-missing.md:48`

**Issue:** Card 2 (line 31) and card 3 (`_lyx/plan/03-gate-test-package-dirs.md:16`) remove only a trailing `/...` before the stat.
`./internal/plan...` is a valid relative Go pattern matching every package whose path starts with `internal/plan`, but after trimming it stays `internal/plan...`, which is no directory, so card 2 raises a blocking `verify-package-missing` and card 3 refuses the gate run.
D3's Bound says "a module-wide pattern … passes" and D4's Bound says "patterns … pass to go test unchanged".
Both cards narrow this to "a module-wide pattern whose stem exists", and that reading only holds for the `/...` spelling.

### [LOW] Card 2 duplicates the chain walk instead of sharing it

**Location:** `_lyx/plan/02-verify-package-missing.md:29`

**Issue:** The card has `missingPackageArgs` walk "exactly as `crossModuleCommands` does", in a second loop.
The cd tracking, the pipeline and `||` scan-end rules and the env-assignment skip then exist twice, and a later fix to one leaves the other wrong, which is also why the focus had to ask whether the copy's branches are tested separately.
With two real call sites the walk is due for extraction under the repo's abstraction rule.
The proposed fix's two Create handles passed `lyx loom validate-plan --overlay` with no finding, the same result as the `{}` baseline.

### [LOW] A cleared read error spends the re-step a following transient failure would get

**Location:** `_lyx/plan/04-loop-status-stop.md:39`

**Issue:** In `runLoop`, `restepped` resets only on `res.ok()`.
Card 4's read error marks the re-step used before re-reading, so when the re-read succeeds and the child that follows fails transiently, the loop takes `LoopStopError` and writes the status failed, where it used to re-step that child once.
D5 licenses this ("under its existing single re-step"), but the card neither states this outcome nor tests it.
On the focus question itself: a status read error meets the re-step either unused or spent by a transient child failure, since the loop reads status again only after a child result and a successful child clears the flag; card 4 states and tests both states (rows at lines 65, 67 and 69).

### [NIT] Re-step rows sit in the interrupted-child test

**Location:** `_lyx/plan/04-loop-status-stop.md:59`

**Issue:** `TestLoop_TransientRestepOnce` (`internal/shedverbs/loop_test.go:424`) is the table for the loop's single re-step.
Card 4 adds "a read error that clears on the re-read steps on", which ends halted with no interruption, and "a read error after a used re-step", which pins how the re-step is shared, as subtests of `TestLoop_InterruptedChildFillsTheErrorForm`.
These rows belong in the re-step table.

## Rejected

- Card 2's `../outside` case (focus item 2): the card reports a result outside the worktree root without a stat and tests it with `go test ../outside`, as D3 says, since such a directory does not exist under the worktree root. The cases that pass unreported are each in D3's Bound or follow from its "cd/-C tracking `verify-nested-module` uses": a variable, `bash -c`, a cd in a pipeline stage, a cd after `||`, an unplaceable cd or -C, an import path, a `.go` file, and a Verify on or after a Move card.
- Verify coverage: every targeted package's build-tagged tests are `integration`, and the overview runs `go test -tags integration ./...`.
- Card 1's `TestPeakContext` arithmetic: `loadCard` adds `package:`/`tests:` per `CardTargetDirs` entry and member glyphs add no file lines, so `startup + (2*3 + 2*4) + 7 + 7` is right.
