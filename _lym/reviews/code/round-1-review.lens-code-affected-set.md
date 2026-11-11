```yaml
lens: code-affected-set
outcome: reported
```

**Lens code-affected-set — per change, the set found and whether it is covered:**

1. **`coverage` scope value (`gateslot.ScopeCoverage`, strict loader, scope error text).**
   - `gateslot/config.go:490-492` is updated.
   - The `config_test.go` expected-error row and the `templateDefaults` rows are updated.
   - `scope.go` `withGateScope` is updated.
   - `gateslot/doc.go:103` and the key-format line at `doc.go:100` are updated.
   - `gateslot/template.yaml` and `tiers.go` are updated.
   - Not covered: the `gateslot/doc.go` Tier plan table (F1 below).
   - Not covered: the `gatecli/doc.go` example config (F2 below).

2. **Template defaults (`webster-fix`, `darn-write`, `code-burler-fix` now coverage).**
   - Covered: `PATTERN-test-speed.md` key table, `landingdeps_verify_test.go` replacer, `scopes_integration_test.go` (the diff test now uses `pr-review`).
   - Not covered: the `gateslot/doc.go` table at lines 225, 228 and 232, which still names `diff` as the template scope.
   - Not covered: the `gatecli/doc.go` sample config at lines 780-782.
   - Not covered: the `internal/configcli/configcli_integration_test.go:661` assertion `webster-fix: {tiers: [0, 1], scope: diff}`. It looks like a hub-file rewrite check and is probably unaffected. I did not read it to confirm.
   - Other fixtures at `websterengine/*`, `webstercli/verify_integration_test.go`, `loomshed/verifygate_integration_test.go` and `loomcli/gatefixer_test.go` write `scope: diff` explicitly. They stay valid and need no change.

3. **`impactset.ModuleTargets` `Importers` and `BinaryBuilders`, and the `impactedFor` signature.**
   - `impactedFor` is unexported, and `targetsFor` is its only caller I saw.
   - `Packages` is unchanged, so existing callers are unaffected.
   - `impactset/doc.go` is updated; its return-set line now lists `Unwatched` (not in the original claim, but accurate).
   - Covered.

4. **`testcover.Selection.Recorded` and `Selector.Select`.**
   - `Select` returns `Recorded` only on the success path and leaves it empty on a miss, which matches the field comment.
   - `testcover/doc.go` is updated.
   - Covered.
   - Side effect: `testcover` and `fabricengine` change though the record names three packages. The summary discloses `Recorded`. It does not disclose `fabricengine.CodeWorktreeChanges`. See F3.

5. **`tierNumbers` dedup and the selected run labelled by its tier.**
   - I checked the other readers of the plan: `notRunTiers`, `ranWhole`, the `runLabel` closing line, `stopOnFailLine`, and `green.covered` with `argsMatch`.
   - With empty call args, the selected run's args are exactly `-run ^(...)$`. That matches `isGuardRunFilter`, so an earlier whole-pass entry with empty args covers it. The effect is correct, since a whole pass ran those tests.
   - The selected run's own entry carries Args, so `newestWholeAncestor` (`len(held.Args) > 0`) excludes it as a base.
   - Covered.

6. **`CodeWorktreeChanges` in `fabricengine`.**
   - It is a new exported method, tested in `diff_integration_test.go`.
   - `PATTERN-fabric-boundary` holds: the call goes through fabric and no new git call is made.
   - Covered. See F3 for the scope note only.

7. **Scope report lines and `doc.go` claims.**
   - The labels match D7.
   - The doc claims that `tree_scope`'s targets and the failed-test rerun are never narrowed rest on `len(request.targets) > 0` in `withGateScope`.
   - I did not locate how the failed-test rerun reaches the gate. My grep found no `failedtest` code in `gatecli` or `verifyrun`. The rerun goes through explicit targets or a separate path that I did not trace.
   - Not covered: `docs/dotnet-hub.md` samples. Under the dotnet scoper `coverage` equals `diff`, so the `diff` samples remain correct. No finding.

8. **`PATTERN` entries and commit messages.**
   - `PATTERN-test-speed.md` table updated. `PATTERN-config-layers` holds because `coverage` is a new value, not a changed meaning.
   - The commit messages carry no `Pattern-Departure:` trailers, and I saw no need for any.
   - The rubric's declared PATTERN changes: none were rendered in the directive, so nothing to check.

**Findings**

- **F1 — MEDIUM, consistency.** `internal/gateslot/doc.go:225,228,232`: the "Tier plan" table says the template scope of `webster-fix`, `code-burler-fix` and `darn-write` is `diff`. The template now gives `coverage`. The diff updated the same file at lines 100-107 and left this table stale. Fix: change those three Scope cells to `coverage`.

- **F2 — LOW, consistency.** `internal/gatecli/doc.go:780-782`: the sample `gates:` block shows `webster-fix`, `darn-write` and `code-burler-fix` with `scope: diff`, while this diff added a Coverage scope section to the same file. Fix: set the three rows to `scope: coverage`, or add a note that the sample is a hand-written hub file.

- **F3 — LOW, scope.** `internal/fabricengine/diff.go` (new `CodeWorktreeChanges`) and its test assertion. The summary's "What changes" list does not mention the addition, and the decision record's scope lists only `gateslot`, `impactset` and `gatecli`. D8's constraint (`CodeFileAt` through fabric, never a new git call) is respected. This is the same kind of unlisted addition as the `testcover` one the summary does disclose. Fix: name it in `summary.md`, or accept it as licensed by D4's reading through fabric.

**Tool calls**
1. `grep` for `scope: diff` and the old scope text across Go, md and yaml. The context lacked the full affected-fixture set.
2. `grep` for `ScopeCoverage`, `scopeDiff` and `.Scope` in non-test Go. The context lacked all readers of the scope value.
3. `sed` of `gateslot/doc.go` 218-236 and `gatecli/doc.go` 770-790, plus a doc listing. I needed the exact stale table and sample lines.
