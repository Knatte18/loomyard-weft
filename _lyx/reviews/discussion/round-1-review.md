---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D2 (guard-directory drop clause and Bound)
    summary: D2's Bound says the guard step skips none, yet D2's own drop clause skips every guard test of a renamed-out or deleted guard package for every card of the batch, and neither the Bound nor Open risks says what still catches those tests
    fix: >-
      In decision-record.md D2, replace the Bound's opening sentence "the step adds tests to the gate and skips none" with a statement that the step skips the guard tests of a dropped guard package directory for the cards of that batch,
      and add what still catches them: a renamed guard test runs in the batch gate when its new directory is a target directory of the renaming card, and in the plan-level verify;
      a deleted guard test runs nowhere, by design.
      Add one Open risks entry naming this skip.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2 (guard-directory drop clause)
    summary: The drop rule removes a whole guard package directory when any one file is renamed out of it, and removes it for every card of the batch, so a batch that moves one file out of cmd/lyx loses all cmd/lyx guard tests at every card gate, including cards that run before the rename
    fix: >-
      In decision-record.md D2, replace the drop clause with a narrower rule:
      a guard package directory drops out of a card's guard step only for that card and the cards after it in the batch, and only when a card at or before it renames out or deletes every Go file of that directory (a directory-level old side or Delete covering it, or file-level ones covering all its files), so a directory that still holds Go files at that card stays in the step.
      Update the Acceptance criteria row "A batch whose card renames or deletes a guard package's files renders a guard step without that directory" to state the narrower rule, and add a row that a card before the renaming card keeps the directory and a batch moving one file out of a guard package keeps it.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D1 (Bound) and Open risks (fifth entry)
    summary: D1's Bound says a file-level old side splits from an earlier card that edited a different file of the same package, but D1's covers rule makes a file-level old side cover only that file, a glyph whose unit is that file, or a package-level ref, so an earlier file-level Edit of a sibling file does not split
    fix: >-
      In decision-record.md D1's Bound, replace "a file-level old side in a package an earlier card touched splits even when the renamed file is not the one that card edited" with "a file-level old side splits from an earlier card whose target is a glyph or handle with that file's package directory as its unit, even when the member lives in another file of that package",
      and reword the fifth Open risks entry to match.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D1 (covers rule, file-level bullet)
    summary: The term package-level glyph or handle can be read as a ref naming the package itself, which would exclude member handles such as plan:internal/pausecli#Foo whose unit is the package directory, the very targets tender batch 07's first acceptance row depends on
    fix: >-
      In decision-record.md D1's file-level bullet, replace "a package-level glyph or handle whose package directory is that file's directory" with "a glyph or `plan:` handle, member or not, whose unit is a package directory equal to that file's directory (`planparser.RefScope` returns that directory with isFile false)".
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3 (Bound) and Open risks
    summary: D3's Bound limits a scan error to that batch's cards, but the cause is a file in the tree that persists into every later batch, nobody is told to fix it, and a walk read error is not the test-file fault the Rationale justifies degrading for, so one stray marker can silently remove guard coverage for the rest of the run
    fix: >-
      In decision-record.md D3, add to the Bound that the error recurs at every later begin-batch until the file is fixed, so guard coverage can be absent for the rest of the run and only the plan-level verify reaches the guard tests then;
      state in D3 that the fork prompt line asks the fork to fix the named file under its card's commit when the file lies in a package its card targets.
      Either give a walk read error its own rationale or move it out of the degrade rule so begin-batch refuses on it with a way forward.
      Add one Open risks entry for a persistent scan error.
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4 (Bound) and Open risks
    summary: D4 lets a fork edit a guard test itself, such as loosening its pinned set, to make the guard step pass, but its Bound does not say that this is what it lets through or what catches a wrong loosening, and Open risks has no entry for it
    fix: >-
      In decision-record.md D4's Bound, state that the instruction lets a fork change a guard test's pinned set or allowlist, that the fork prompt tells it to fix the code that broke the guard where that is the right fix and to change the pin only when the pin itself must move,
      and that such an edit stays visible as a scope-guard deviation in the batch record and the batch's review.
      Add one Open risks entry naming a fork that loosens a guard to pass its gate.
  - id: F7
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D2 (directory walk sentence)
    summary: The walk takes every directory holding a _test.go file except under a nested go.mod, while go list also skips testdata and directories whose names start with an underscore or a dot, so a testdata fixture test file outside a nested module would be scanned, and a deliberately malformed one would trip D3 and drop the guard step for the run
    fix: >-
      In decision-record.md D2, change the walk sentence to say the walk skips `testdata` directories and directories whose names begin with `_` or `.`, as `go list` does, as well as any directory under a nested `go.mod`.
      Add an Acceptance criteria row that a `_test.go` file under a `testdata` directory is never scanned.
  - id: F8
    severity: NIT
    class: decision
    location: _lyx/discussion/decision-record.md D2 and D3 (Rationale)
    summary: D2's Rationale compares against a plan-time check and D3's against refusing begin-batch, both rejected alternatives already recorded in support-log.md, which the stencil keeps out of Decisions
    fix: >-
      In decision-record.md, delete D2's Rationale sentence beginning "A runtime catch is generic across hubs", and in D3's Rationale replace "refusing would stall the batch on a fault the plan-level verify already reaches" with "the plan-level verify already reaches the fault".
      Both points already stand in support-log.md's Rejected alternatives.
  - id: F9
    severity: NIT
    class: consistency
    location: _lyx/discussion/support-log.md Question ledger
    summary: Q7 is listed before Q6
    fix: In support-log.md's Question ledger, move the Q6 bullet above the Q7 bullet.
---

### [MEDIUM] D2's Bound claims the guard step skips nothing while its drop clause skips guard tests

**Location:** `_lyx/discussion/decision-record.md` D2

**Issue:**
D2 says a guard package directory a batch card renames files out of or deletes "drops out of the step for every card of that batch".
The Bound then opens with "the step adds tests to the gate and skips none".
The drop clause does skip tests: every marked test of that package, at every card gate of the batch.
The Bound and Open risks never say what still catches them.
In `internal/websterengine/cardgate.go` the batch gate runs `lyx gate test --tags integration` over the union of the batch cards' target directories, and `planparser.CardTargetDirs` takes a Rename group's new side, so a renamed guard test runs again in the batch gate.
A deleted one runs nowhere.
The plan writer needs that stated to write the acceptance test and the doc.

### [MEDIUM] The guard-directory drop is wider than the problem it solves

**Location:** `_lyx/discussion/decision-record.md` D2

**Issue:**
The drop exists so a card gate never names a directory that is gone by the time that card's gate runs.
As written it goes further in two ways.
First, by D1's covers relation, a file-level old side covers a package-level target in that file's directory, so renaming one helper file out of `cmd/lyx` drops `cmd/lyx` from the guard step, though `cmd/lyx` still holds most of the guard tests (29 of the 42 marked files counted with `grep -rln '^//lyx:guard'`).
Those are the repo-scanning tests from #594 that D2 exists to run.
Second, the drop applies to every card of the batch, including cards that run before the renaming card, when the directory still stands.
Both widen the skip in F1 for no gain.

### [LOW] D1's Bound describes a split its covers rule does not produce

**Location:** `_lyx/discussion/decision-record.md` D1 Bound and Open risks

**Issue:**
The Bound says "a file-level old side in a package an earlier card touched splits even when the renamed file is not the one that card edited".
Under the covers rule, a file-level old side covers an earlier target that is that file, a glyph whose unit is that file, or a package-level ref in that directory.
An earlier card that edited a sibling file by file path is none of these, so it does not split.
The over-split comes from package-unit glyphs and handles only.
The Open risks entry repeats the Bound's wording.

### [LOW] "Package-level glyph or handle" is ambiguous on the case the first acceptance row needs

**Location:** `_lyx/discussion/decision-record.md` D1

**Issue:**
The support log records tender batch 07's done-check findings on `plan:internal/pausecli#…` and `plan:internal/pauseengine#…` handles.
`planparser.RefScope` gives such a member handle its package directory with `isFile` false.
"Package-level" reads most naturally as a ref naming the package itself, not a member in a package unit.
A plan writer taking that reading would not split card 12 from member-handle targets, and the first acceptance row would fail.

### [MEDIUM] D3's bound understates how long a scan error removes guard coverage

**Location:** `_lyx/discussion/decision-record.md` D3

**Issue:**
D3's Bound says a scan error "drops only the guard step for that batch's cards".
The scan reads the tree, so a misplaced marker or unparsable test file fails the scan again at each later begin-batch until someone fixes it.
The prompt line names the error but tells nobody to fix it.
A fork that commits a stray `//lyx:guard` turns the guard step off for the rest of the run.
D3 also degrades on "a read error in the walk", but its Rationale only covers a fault in a test file.
`internal/impactset/guards.go` separates `GuardScanError`, the author-fixable kind, from read failures for exactly this reason.
Open risks says nothing about either.

### [MEDIUM] D4 does not bound a fork loosening a guard

**Location:** `_lyx/discussion/decision-record.md` D4

**Issue:**
D4 tells the fork to fix a guard step failure outside its card, citing "the pinned set in a `cmd/lyx` test".
That edit can change the guard itself, adding the fork's new call site to an allowlist so the guard passes.
That is the right fix for some #594-shaped breaks and the wrong one for others.
The Bound covers only which files are in scope ("a file the guard step's failure names").
It does not say that this lets a fork weaken a guard, what prompt wording steers it to the code fix first, or what catches a wrong loosening.
The rubric's attack-surface question requires that, and Open risks has no matching entry.

### [LOW] The directory walk does not match `go list`'s package set

**Location:** `_lyx/discussion/decision-record.md` D2

**Issue:**
D2 replaces impactset's `go list` with a walk "for directories holding `_test.go` files, skipping any directory under a nested `go.mod`".
`go list ./...` also skips `testdata` and `_`/`.`-prefixed directories.
Today's only `testdata` `_test.go` files sit under `internal/planglyph/testdata/glyphchain/go.mod`, so the nested-module skip happens to cover them.
A future fixture without its own `go.mod` would be parsed by the scan, and a deliberately misplaced-marker fixture would fail it and disable the guard step for the whole run under D3.
It would also hand `lyx gate test` a `testdata` path as a package.

### [NIT] Rejected-alternative comparisons in D2 and D3 Rationale

**Location:** `_lyx/discussion/decision-record.md` D2, D3

**Issue:**
D2's last Rationale sentence argues against a plan-time check, and D3's Rationale argues against refusing begin-batch.
Both are rejected alternatives the support log already records.
The stencil keeps rejected alternatives out of `## Decisions`.
No requirement is lost by cutting them: D3's no-refusal rule is stated in its Decision text.

### [NIT] Question ledger order

**Location:** `_lyx/discussion/support-log.md` Question ledger

**Issue:**
Q7 is listed before Q6.

## Focus notes

- Focus 2 (D1 covers against acceptance rows): the rows "a later card that only edits the same package does not split" and "a card never splits from itself" agree with D1, since an Edit is never an old side and D1 requires a distinct earlier card.
  The only disagreements found are F3 and F4.
- Focus 4 (traceability and relocation): every Scope In item and acceptance row traces to a Decision or Constraint.
  The only relocation candidates are F8's two rationale sentences, and neither holds a requirement.
- Focus 5 (support log against record): every lens-review fix marked fixed in Interview Turn 3 appears in the decision record as described, and every ledger resolution matches its Decision.
