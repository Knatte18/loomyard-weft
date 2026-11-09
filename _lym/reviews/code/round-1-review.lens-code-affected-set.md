```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. For each change, the set I found and whether the diff covers it:**

- **Old VS Code title form (`ToUpper(shortname)` + `: ` + slug):** covered.
  - `ideengine.lookFor` and `fabricengine.workspaceWindowTitle` were the only sites that formed it.
  - Both now call `boardengine.IDEWindowTitle` or `consoletitle.Workspace`.
  - A repo-wide grep finds no remaining `ToUpper(shortname)`.
  - Tests and fixtures that asserted `TST:` were lowercased: `look_test.go`, both `ideengine` integration tests and `hubworkspace_integration_test.go`.
  - `docs/overview.md`, the `lym ide` paragraph and `fabricengine/doc.go` were updated.
  - The `internal/vscode` tests use `LY:` strings only as inputs to a splice, not as asserted output, so they stay correct.
- **Terminal title default (`filepath.Base(dir)`) in `lym reed window`:** covered.
  - The code, the verb's long help and example, the flag help, `reedcli/doc.go` and the `window_test.go` rows are updated.
  - The only other callers of `lym reed window` are the three reed template button groups (posix and windows) and the batten `open_window_command`.
  - The buttons drop `--title {slug}`; batten keeps `--title {title}`, which now resolves to the console title.
  - `reedengine/config_test.go` is updated.
  - No stencil, contract or plugin file mentions `reed window` or the old title.
- **Batten `{title}` placeholder (`TitleWithHandle(...)` to `TerminalWindowTitle`):** covered except the points below.
  - Updated: `wire.go` comment, `battenshed/doc.go`, `battenshed/template.yaml` comment, `docs/overview.md` line 1586.
  - Not updated: `commanddeps_test.go`. See findings 1 and 2.
- **`TitleWithHandle` callers (`orchcli/runs.go`, `loomcli/sharedbootstrap.go`):** left as they were.
  - The function is not dead.
  - These two still form a tmux window name (`beam: <slug>`, `<handle>: <recipe>`) without `consoletitle`. See finding 3.
  - `docs/overview.md:1059` still says `T4: tapestry`. That matches the unchanged status window.
- **New `internal/consoletitle` leaf:** covered.
  - `PATTERN-leaf-packages.md` has the entry.
  - `docs/overview.md` has the package line.
  - The package has `doc.go`, a leaf enforcement test (`scankit.AssertImportAllowlist`) and a `tmuxkit.Main` test main, like the `durfmt` and `machinename` siblings.
  - `PATTERN.md` lists no per-package leaves, so it needs no edit.
  - The diff declares no PATTERN changes beyond this entry, and there are no `Pattern-Departure:` trailers.
- **New import edges:**
  - `boardengine` to `fabricengine` already existed (`board.go`, `config.go` and others), so it is not new.
  - `fabricengine` to `consoletitle` is stdlib-only and allowed by the new entry.
  - `reedcli` to `boardengine` is new. I found no guard test or PATTERN entry forbidding it.
  - `PATTERN-fabric-boundary` is unaffected: fabric gains only the leaf import and learns nothing of the board.
- **Readers of a title:** none found.
  - `shedadapters` passes `{title}` through as a value only.
  - `desktophook` and `termwindow` receive the title as an argument.
  - The summary's "no reader" claim holds.
  - It does not name the `docs/overview.md:1059` window or the `runs` window, which are unchanged. This is a gap in the landing note, not a defect.

**Findings:**

1. **LOW, consistency.** `internal/battencli/commanddeps_test.go:15`, `:26-28`.
   - The test and its name (`TitleOpensTheSlugWithTheTasksHandle`) still describe the old `T4: run` form.
   - The comment now claims the console title, but no row sets a hub shortname or a project handle, so `commandDeps`' new call to `TerminalWindowTitle` is never exercised with those.
   - `TestWindowTitles` covers the function itself. The wiring from `commandDeps` to it is what's unpinned: `location.WorktreeName` as `primeName`, and `slug`.
   - Evidence: `commanddeps_test.go:1-2` and `:26-28` against `wire.go:996`.

2. **LOW, consistency.** `internal/battencli/commanddeps_test.go:1-2`, `wire.go:980-982`.
   - Both comments say the title falls back to "the bare slug when the board holds no handle". With a shortname recorded, the title is `<shortname>: <slug>`, not the bare slug.
   - That is true only for a hub with no shortname, as in the test fixture.

3. **MEDIUM, scope.**
   - **Location:** `internal/orchcli/runs.go:51-57` and `internal/loomcli/sharedbootstrap.go:381-383`.
   - **Finding:** the Goal's Limits say "the places that form a title today call it" (one shared function).
     - These two places still form a window title through `TitleWithHandle`, with no hub shortname and no `<P#>`.
     - They are tmux window names, the same ones the Goal's Seen section complains about ("carry names with no hub shortname").
   - **Why it matters:**
     - The summary justifies leaving them by reading "the `runs` strand … no title of their own" as covering the runs window and the status window.
     - That reading is a design call. The status window `T4: <recipe>` is not the `runs` strand, so the call is open to challenge.
     - If the call stands, the Limits' "the places … call it" is only partly met, and the summary should say why.
   - **Evidence:** `runs.go:56`, `sharedbootstrap.go:382`, the decision record's Goal (Seen and Limits), and the summary's "The tmux windows inside a session keep their names".

**Tool calls:**
1. Grep of docs, contracts, plugins and `doc.go` files for old title forms. My context held no search of the doc and stencil roots.
2. Grep for other references to `durfmt` and `machinename`, and `PATTERN.md`, to see whether leaf packages are listed elsewhere.
3. Grep for import-layer tests on `reedcli` and `boardengine`, plus the `shedadapters` `Title` doc, to check `reedcli` to `boardengine` is not forbidden.
4. Grep for docs describing `lym reed window`, to confirm no spec still states the old default title.
