---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/plan/02-verify-package-missing.md:31
    summary: Card 2 lets a package argument that resolves outside the worktree root pass, an exemption D3 neither decides nor lists in its bound
    fix: In card 2's Intent, delete the clause "; a result outside the worktree root (`..` or `../…`) passes" so such an argument is checked like any other and reported, since it names no directory under the worktree root and none a card creates, as D3 states. Add the reporting row "`go test ../outside` in the overview chain, with an empty card" to `TestValidate_VerifyPackageMissing`'s rows that report. The fix changes no target and no verify command.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/02-verify-package-missing.md:29
    summary: The scan-end branches of card 2's missingPackageArgs walk (cd in a pipeline stage, cd after ||, a -C that cannot be placed) have no named test input, and only the unplaceable-cd branch is reached by a row
    fix: "In card 2's Tests, add three rows to the rows that report nothing in `TestValidate_VerifyPackageMissing`: `true | cd internal && go test ./nope` (a cd in a pipeline stage ends the scan), `false || cd internal && go test ./nope` (a cd after || ends the scan) and `go -C .. test ./nope` (a -C that cannot be placed ends the scan). Each would report `./nope` if the scan went on, so each row reaches its branch. The fix changes no target and no verify command."
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/04-loop-status-stop.md:39
    summary: Card 4 says what a status read error does when the loop's one re-step is unused, but not when an earlier transient child failure has already used it, so that branch has neither a stated outcome nor a test
    fix: "In card 4's read-error bullet, add: When the re-step is already used, it stops at once with `status-unreadable: <error>` and the read-error way forward, with no re-read. Add the subtest a read error after a used re-step stops without a re-read to `TestLoop_InterruptedChildFillsTheErrorForm`: the first child returns a transient error envelope, `fx.spec.Loop.ReadStatus` then returns `errors.New(\"read lock unavailable\")`, and the subtest asserts one child started, an error holding `status-unreadable: read lock unavailable`, and `loop.restep` naming the child's transient error rather than the read error."
---

### [MEDIUM] Card 2 exempts arguments outside the worktree root without a decision

**Location:** `_lyx/plan/02-verify-package-missing.md:31`

**Issue:** D3 makes a relative argument a finding when its directory "neither exists on disk under the worktree root" nor is or contains a created directory, and it lists `..` and `../…` among the spellings it reads.
D3's bound lists the arguments that pass (variable, `bash -c`, after an unplaceable `cd`, import path, module-wide pattern, `.go` file, Verify on or after a Move, a directory an earlier card deletes), and an argument outside the root is not among them.
Card 2 adds "a result outside the worktree root (`..` or `../…`) passes", so `go test ../sibling` in the overview chain goes unreported where D3 reports it.
The card's own Bound paragraph does not mention this pass either, so it is an unbounded exemption with no license in the answer key.

### [MEDIUM] Card 2's copied chain walk has unreached scan-end branches

**Location:** `_lyx/plan/02-verify-package-missing.md:29`

**Issue:** `missingPackageArgs` walks a chain "exactly as `crossModuleCommands` does" but as its own loop: `crossModuleCommands` in `internal/planparser/verifymodule.go:167-206` keeps the walk inline, and the card neither extracts it nor reuses it.
So the copy's guard branches are not reached by `TestValidate_VerifyNestedModule`'s rows, which reach only `crossModuleCommands`'s copy.
Card 2 names a row for the unplaceable `cd` (`cd $DIR && go test ./nope`) but none for the scan end at a `cd` in a pipeline stage, a `cd` after `||`, or a `go -C` that cannot be placed.
A copy that dropped any of those ends would report a spurious finding that no planned row would catch.

### [LOW] Card 4 does not say what a read error does when the re-step is already spent

**Location:** `_lyx/plan/04-loop-status-stop.md:39`

**Issue:** `runLoop` keeps one `restepped` flag (`internal/shedverbs/loop.go:244-249`): a transient child failure sets it, and only a success envelope clears it.
Card 4 routes a read error through that same re-step "when the loop's one re-step is unused" and does not say what happens otherwise.
That case arises when a transient child failure is re-stepped and the next iteration's status read fails.
D5 implies a stop with `status-unreadable: <error>`, but the card leaves the implementer to infer it, and no subtest reaches it.

## Rejected

- Card 1's test targets (focus 1): `internal/websterengine/cardgate_test.go` is `package websterengine`, an internal test package, so `internal/websterengine#TestCardGateCommand` is the right member glyph.
  `internal/batcher/estimate_test.go`, `internal/planparser/targetdirs_test.go` and `internal/planparser/validate_test.go` are external `_test` packages.
  Naming them by file path is what the overview's Shared Decision prescribes, and it holds: `targetDir` (`internal/planparser/targetdirs.go:85-92`) passes a `_test` unit through unchanged, so the running binary would derive `./internal/planparser_test` from a member glyph.
  Not a defect.
- Re-sign arrows (focus 1): the signature changes are `goCommandDirAndPackages` on card 2 and `readStatusAt` on card 4, and both carry the arrow.
  The new `LoopSpec.ReadStatus` and the `childResult` fields are struct fields, which take none.
- Card 3's refusal (focus 2): `missingPackageDirs` runs after `absoluteDir` and before `catchTerminatingSignals` and `resolveHub` (`internal/gatecli/run.go:43-51`), so no upstream check rejects `./internal/x_test` or `./nope` first.
  The existing `-C directory does not exist` row is still refused by `absoluteDir` first, as intended.
  The named rows reach every clause, and the `TestGateTest_Scenario` rows pin the in-hub ordering.
  The card also creates the `pkg`, `held`, `nested` and `enclosed` directories that the scenario's existing arguments (`internal/gatecli/test_integration_test.go:282-656`) would otherwise now trip.
- Card 4's branches (focus 3): absent file, decode error (`state.ReadJSONNoMkdir` wraps `state.ErrDecode`, `internal/state/state.go:178`), a read error that clears on the re-read and a read error that fails it each have a named subtest.
  The refusal-spec `step interrupted` row and the new `loop status missing or undecodable` row are both carried.
  `shedverbs.Spec.MissingStatusWayForward` is set by loom and batten (`internal/loomcli/arm.go:290`, `internal/battencli/arm.go:362`), so the told reseed clause reaches the stop.
  The skipped `WriteFailedStop` is bounded to status-read stops and stated as the departure from `PATTERN-failure-stop-visible`.
- Fidelity (focus 4): D1 to D7 and every Constraint are carried.
  `internal/batcher/doc.go:39` names "each directory the card's targets live in", not test directories, so it needs no edit.
  `internal/planparser` sits at its budget row of 92 top-level tests, and card 2 lists `cmd/lyx/testdata/test-budget.yaml`.
  The other cards add rows or subtests only.
  Card 5's D6 commit line is licensed by D6.
  `lens-review.md` is the sanctioned non-card file (`planparser.LensReviewFileName`, `internal/planparser/amendment.go:24`), not a card.
- Card order and `## verify:` (focus 5): card 2's `createdDirs` relies on card 1's mapping and follows it, and cards 3 to 5 depend on no later card.
  The chain runs `go test ./...` and `go test -tags integration ./...` in the template's order, covering every targeted package.
  `lyx loom validate-plan --overlay` with `{}` reports the plan clean.
- Test economy (focus 6): `TestValidate_VerifyPackageMissing` is the first test of a new check, and no existing table reads `verify-package-missing`.
  The `TestCardGateCommand` and `TestPeakContext` rows pin the acceptance criterion at the two readers it names, where no existing row covers a `_test` unit.
