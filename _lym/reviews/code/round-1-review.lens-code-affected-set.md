```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff's changes force outside the lines they touch.

**Per-change list**

1. **No-test line (`noTestNotice`, new `notice` parameter of `runInvocations`)**
   - Set found:
     - the caller `runTier`;
     - the test caller `testtargets_test.go`, updated;
     - `doc.go` lines 570–575, updated;
     - the two stencils;
     - the shell guard's refuse-raw row (`shellguard/rows_test.go`, `claudeengine/doc.go` 545–585);
     - `PATTERN-refusal-way-forward`;
     - `PATTERN-test-speed`.
   - Covered: the caller, the test caller and `doc.go`.
   - Not covered: the refuse-raw interaction (finding 1) and the sibling stencil family (finding 2).

2. **Records-to-code mapping (`CodeLocation`, `resolveHub`)**
   - Set found:
     - the eight `resolveHub` callers (`detachfront.go`, `cardfixer.go` twice, `report.go`, `run.go`, `testmap.go`, `status.go`, `redundant.go`, `cardverify.go`);
     - `doc.go` lines 412–416 and the testmap refusal sentence, both updated;
     - `hubgeom.PrimeWorktreePath`, whose only caller is `testmap.go`;
     - `PATTERN-cwd-resolution`.
   - Covered:
     - the `doc.go` text;
     - the refusal test rows;
     - `PrimeWorktreePath` needs no change.
   - Not covered: `PATTERN-cwd-resolution` (finding 3).

3. **Flaky tests:** only the test files changed, and no production set is affected. Covered.

4. **Declared PATTERN changes:** the directive declares none, and `PATTERN.md` and `pattern/` are untouched. Nothing to raise.

5. **Commit messages and departures:** none of the three commits carries a `Pattern-Departure:` trailer. A departure for finding 1 would have needed one, but see finding 1.

**Findings**

1. **MEDIUM, design.** The new stencil text and the no-test line's way forward tell agents to run a raw `dotnet test`. A hub that lists `dotnet test` in `gate.yaml`'s `refuse_raw` has the shell guard refuse that command. The refusal reads "a raw go test or dotnet test run is refused here".
   - Evidence: `contracts/stencils/darn/darn-fragment-make-change.md:15-16`, `contracts/stencils/webster/webster-body-implementer.md:117-118`, `internal/gatecli/tierrun.go:377`, `internal/shellguard/rows_test.go:67`.
   - The rule it runs against is `PATTERN-refusal-way-forward`, which says no refusal or stencil names a step agents' settings deny. `claudeengine/doc.go:558` shows the repository's precedent: steer text names a step only while `refuse_raw` does not refuse it.
   - The edited text offers no route for a hub that refuses it, and it contradicts the sentence "never a raw test command" in the same stencil. I could not confirm that any hub lists `dotnet test`; the shell-guard test only shows that a hub can.

2. **MEDIUM, consistency.** Two documents state the same "never a raw test command" rule: `contracts/specs/loom-plan-spec.md:352` and `contracts/stencils/loom/loom-block-plan-cards.md:63`. They are about card `**Verify:**` lines and are not changed by the diff. That is consistent, since the exception is for hand runs. I found no stencil in the writer or implementer family that was missed, so this is **not a finding**. I list it only to record that the set was checked and is covered.

3. **LOW, consistency.** `pattern/PATTERN-cwd-resolution.md` says only `lymcwd` resolves cwd or worktree roots and that other code gets another worktree's `Location` through `lymcwd.NamedWorktree`. `fabricengine.CodeLocation` does use `NamedWorktree`, and `fabricengine` is the permitted caller, so it complies. A grep for `PrimeWorktreePath` and `code worktree` in `PATTERN.md` and that file found nothing, so no pattern text needs updating. Not a defect.

Only finding 1 is a real finding. Finding 2 is a confirmed non-issue and finding 3 is a compliance note.

**Tool calls**
1. `grep` across `contracts`, `plugins`, `internal` and `docs` for the raw-test-command wording, `doc.go` records-worktree text, and `PrimeWorktreePath` in `PATTERN.md` and the cwd pattern: my context lacked the sibling stencils and specs that state the same rule.
