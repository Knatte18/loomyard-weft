```yaml
lens: code-affected-set
outcome: reported
```

Task: list, per change, the set of dependents outside the touched lines, and report any member the diff leaves wrong (lens code-affected-set).

**Per-change list**

- **`HEDDLE.md` joins the walk's hub-wide commit** (`walk.go:264-266`).
  - Set found: tests that assert the walk's board commit contents (`ensure_integration_test.go:590`, `:759`, `:1457`, `:1530`; `reinterpret_integration_test.go:102`), plus the heddle CLI test that checks Bolt status.
  - Covered, except `:1530` (finding 1). The `:759`, `:1457` and `reinterpret_integration_test.go:102` assertions only check that expected paths are present, so an extra file does not break them.
  - The heddle CLI test is covered by the diff's edit at `cli_integration_test.go:504-507`.
- **`hubreconcile` now imports `heddleengine`.**
  - Set found: the import allowlist (`seam_enforcement_test.go:22`) and `PATTERN-leaf-packages` / `PATTERN-config-layers`. The config-layers entry lists `heddleengine` only as a degrading config caller; it does not constrain `hubreconcile`.
  - Covered. The allowlist is updated.
  - `heddleengine`'s dependency closure does not include `hubreconcile`, so there is no cycle.
- **A new writer of the overview.**
  - Set found: `pattern/PATTERN-heddle-instructions.md:34-36` (updated), `internal/heddleengine/doc.go` (updated), `internal/hubreconcile/doc.go` (updated), and `docs/overview.md:344`.
  - Covered. `docs/overview.md:344` describes the file generically and names no writers, so it stays correct.
  - No stencil, spec or other `.md` names the overview's writers.
  - `PATTERN.md` carries no declared change for this entry.
- **`snapshotConfig` refactored to `snapshotFile`.**
  - Set found: the only callers are `snapshotConfig` (`walk.go:530`) and the new overview snapshot (`walk.go:228`).
  - Covered. Behavior is unchanged.
- **Warn log call** (`walk.go:298-299`).
  - Follows the package's `way_forward` convention (`hubreconcile.go:63`, `walk.go:241`).
- **Heddle CLI test expects `" M HEDDLE.md"` after a set.**
  - Covered by the diff's edit at `cli_integration_test.go:504-507`.
- **Commit messages.**
  - The two commits carry no `Pattern-Departure:` trailer, and none is needed.

**Findings**

1. **MEDIUM, class `consistency`.** `arrangeUnpushedHubWide` at `internal/hubreconcile/ensure_integration_test.go:1529-1536` may now fail.
   - The helper filters the walk's commit down to files other than reinterpretation markers and `.template-base` files, then requires `len(changed) == 1`, "one config file".
   - The walk now also commits `HEDDLE.md` whenever the hub fixture's board lacks it or holds a stale one. The new test seeds exactly that case. Nothing in the repo tracks a `HEDDLE.md` fixture.
   - If the `hubforge` fixture board has no up-to-date overview, `changed` gains a second entry and this helper's `t.Fatalf` fires. It feeds the callers of `unpushedHubWide`, including `TestEnsure_UnpushedHubWideCommitOverMovedUpstreamLeavesStampAbsent`.
   - I could not confirm whether the fixture holds an overview, so this is unverified. If the gate passed, the fixture board holds a current overview and the finding is void.
   - Evidence: `ensure_integration_test.go:1529-1536` against `walk.go:264-266`.

**Tool calls**
1. `grep` for `HEDDLE.md` / `RenderOverview` in `.md`, `.tmpl`, `.yaml`, `.json` files, to find docs and stencils naming the overview's writers. My context lacked that.
2. `grep` for tests asserting the walk's board commit, and for heddle mentions in `PATTERN-config-layers` and `PATTERN-fabric-boundary`.
3. `grep` for `--name-only` and related commit assertions in the `hubreconcile` tests.
4. Read `reinterpret_integration_test.go:95-115` and `ensure_integration_test.go:1452-1470` and `:1525-1540`, to see how each test filters the committed files.
5. Read `ensure_integration_test.go:1495-1524` and run `git ls-files | grep -i heddle.md`, to see the `arrangeUnpushedHubWide` setup and whether any `HEDDLE.md` fixture is tracked.
