```yaml
lens: code-affected-set
outcome: reported
```

**List: per change, the set found, covered or not**

1. **Warp exclude default gains `.claude/settings.local.json`** (`template.yaml`).
   - Set: `template_test.go`, `warpexclude_integration_test.go`, `render_test.go` fixture, `seam_integration_test.go` and `fabricengine/doc.go` are all updated.
   - Other readers of the literal list: the grep found none.
   - Covered.

2. **Reconcile rewrite for hubs with their own `warp_exclude`** (`configreg.go:156`, `AppendExcludePaths`, `SettingsExcludeReinterpretation`).
   - Set: the registry row, `fabricengine/doc.go` and `warpexclude_test.go`.
   - Other reinterpretations have an integration test: `hubreconcile/reinterpret_integration_test.go:54` exercises `driftengine.DirBaseReinterpretation` through the real reconcile. The new `heddle-claude-settings-excluded` marker has no counterpart. `warpexclude_test.go` covers only the rewrite function, not the registry row or the once-per-hub marker (finding 1, LOW).

3. **`writeRendered` stages by target name, not file name** (`render.go:236`).
   - Set: the only callers are `render.go` and `settings.go`.
   - The `render_test.go` leftover-temp row is updated to `claude-file.tmp`.
   - Doc comments are updated.
   - Covered.

4. **New `claude-settings` target and `JSONObject` field** (`target.go`).
   - Set: `ValidateEntry` (`store.go`), `SetOverride` (`override.go`), `Render`, `Show`, the `heddlecli` help and `doc.go`, `heddleengine/doc.go`, `docs/overview.md:343` and `PATTERN-heddle-instructions.md`.
   - Seam `Made` renders every target, so the prime at `fabric clone` and an added worktree both get the file.
   - Seam `TookIn` selects targets by `ReRender`. The new target has no `ReRender`, so a take-in never re-renders it. That is consistent with the record, which only asks for rendering at worktree creation and clone.
   - Covered.

5. **Shipped layer gains three `claude-settings` entries** (`shipped.json`).
   - Set: `store_test.go` `ShippedEntriesArePinned` is updated.
   - The grep for the other shipped slugs found only `heddlecli/verbs.go` examples, which are unaffected.
   - Covered.

6. **`.claude/settings.json` removed.**
   - Set: the grep found no remaining reference outside `_lym`, `.lym` and research docs.
   - Covered.

7. **Rule documents.**
   - `pattern/PATTERN-heddle-instructions.md` gains the two bullets and `docs/overview.md` is updated.
   - The constraints section lists no declared PATTERN changes, so there is nothing further to check there.
   - Covered.

8. **Commit messages.**
   - Two commits, neither carries a `Pattern-Departure:` trailer.
   - The `PATTERN-fabric-boundary` entry says fabric "learns nothing of IDEs, shuttle, the board or heddle". `configreg.go` passes `heddleengine.ClaudeSettingsFile` into `fabricengine.AppendExcludePaths`, so fabric itself names no heddle path. Fine.
   - `settings.go` in heddle imports `fabricengine`, as `render.go` already did.
   - No finding.

**Findings**

- **F1, LOW, `scope`.**
  - Location: `internal/configreg/configreg.go:156`.
  - The new fabric reinterpretation, `heddle-claude-settings-excluded`, is registered with no reconcile-level test. `internal/hubreconcile/reinterpret_integration_test.go:54` shows the existing pattern: an integration test drives a hub with its own list through the real reconcile and checks the marker.
  - Only the pure rewrite function is tested in `warpexclude_test.go`, so a wrong registry wiring or ID would pass every test.
  - This is borderline against "test economy". It is a behavior the diff adds (a hub with its own `warp_exclude` gaining the path through the reconcile), and the public-surface path for it is untested.

**Tool calls**
1. Bash grep for the exclude-list literal, `.claude/settings.json` references, target-name lists and `.tmp` naming: to find consumers outside the manifest.
2. Bash grep of `docs`/`README` for heddle and render mentions, other users of `Report`, and the heddleengine file list: to check docs and downstream consumers.
3. Bash grep for the shipped slugs, the other reinterpretation IDs and `warp_exclude` outside fabric: to find fixtures and reinterpretation tests.
