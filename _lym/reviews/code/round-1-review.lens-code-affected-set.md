```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. For each change, what reads or asserts it, and whether the diff leaves it wrong.**

Per change, with the set I found:
- **`beamHub` now calls `CopyRegistered`.**
  - There are five callers, all in `internal/loomcli/projectrecord_integration_test.go`, at lines 72, 91, 123, 140 and 156.
  - A test in `internal/loomcli` takes `*hubforge.Hub` and `location` as before, so no signature changed. Covered.
  - Each caller mutates only its own copy; the `os.Remove` of the shortname record at line 141 hits the copy. Covered.
  - Tests with their own `Add` calls do not use `beamHub`, so nothing is added twice. Covered.
  - `internal/battencli` has its own `beamHub`, which is outside this diff, so there is no cross-package caller. Covered.
- **`newBadReedUpFixture` now calls `CopyRegistered`.**
  - There are four callers, in `smoke_prebootstrap_integration_test.go` at lines 200, 292, 321 and 329.
  - The signature is unchanged and `seed` still runs after `lymcwd.Resolve`. Covered.
  - `newWiredPairFixture` (`smoke_helpers_test.go:63`) is unchanged and still has about 12 callers on `CopyHub`.
  - It keeps its own literal `"loom-smoke-task"` (line 75), the same value as the new `badReedUpSlug`. Only a few lines of duplication, so no finding.
- **New registration keys `loomcli-beam-hub` and `loomcli-bad-reed-up`.**
  - Only these two non-`hubforge` `Registration{` literals exist in the tree.
  - Shape names have the form `<anchor>+<slugs>`, so they cannot collide with these keys. Covered.
- **Rule documents.**
  - The `PATTERN-test-isolation` hub-fixtures section (`pattern/PATTERN-test-isolation.md`, lines 186–208) already describes registered templates, and the diff adds no wrapper type. Covered.
  - `PATTERN-test-speed` is unaffected, since tier calls and `t.Parallel` are unchanged. Covered.
  - There is no `Pattern-Departure:` trailer, and none is needed. The commit messages match the diff.
- **Seams.**
  - `TestRunOwnerVerdict` (`ownercheck_integration_test.go:33`) installs `boardkit.InstallReplay`, but it uses `CopyHub` and not either registration.
  - No test using `beamHub` or `newBadReedUpFixture` installs a seam or calls `Setenv`. Covered.
- **Change description.**
  - The four-site claim for `newBadReedUpFixture` holds.
  - The two-site claim for `newWiredPairFixture` is wrong. It has about 12 callers, so the real reason it stays on `CopyHub` is not "below the three-site threshold" (see F1).
  - The six-site claim for `beamHub` is wrong. There are five call sites (see F2).

**Findings**

- **F1 — LOW, consistency.**
  - Location: `_lym/landing/summary.md`, "Design decisions".
  - Summary: the change description says `newWiredPairFixture` "has two sites, below the three-site threshold", but it has many callers.
  - Evidence: callers at `smoke_test.go:354` and `:498`, `smoke_drivingsurface_test.go:119`, `:189` and `:269`, `smoke_prebootstrap_integration_test.go:39` and `:152`, and more.
  - The count of two might mean sites with identical build steps, but the text doesn't say so, and the claim is not borne out as written.
  - Fix: reword the description to give the real reason, or correct the count.
- **F2 — LOW, consistency.**
  - Location: `_lym/landing/summary.md`, `beamHub` bullet ("six tests and subtests").
  - Summary: grep shows five `beamHub` call sites, at `projectrecord_integration_test.go:72`, `:91`, `:123`, `:140` and `:156`.
  - Fix: correct the count to five, or say what the six are.

Tool calls: none. The manifest already held every definition and call-site list I needed.
