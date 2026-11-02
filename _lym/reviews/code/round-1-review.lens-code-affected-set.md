```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens for the `ace46fb8` diff, listing for each change what it forces outside the lines it touches.

**Per-change set**

1. **`RunButton` gains the `anchorOf` parameter.**
   - Set: the one production caller `internal/reedcli/button.go:77` (updated), `buttonDeps` literals in `buttonclick_test.go` (updated) and in `buttonclick_integration_test.go` (updated).
   - No other caller of `RunButton` exists in the repository. Covered.

2. **`runButtonVia` now runs in `runDir`, not `spec.Dir`.**
   - Set: `composeConfirmedButtonLine` (`internal/reedengine/panebin.go:167`) and its two direct tests (`buttonclick_test.go:40` and `:235`).
   - The function only takes a `dir` argument, so no signature change is forced. Covered.
   - The "could not raise its prompt" message now names `runDir`, and the message tests cover it.

3. **New "anchor does not resolve" message and log lines.**
   - The `TestRunButtonVia` row `UnresolvableAnchorRunsNothing` asserts the message.
   - The docs list the new check in the order of checks: `reedengine/doc.go:87` and the `runButtonVia` doc comment. Covered.

4. **Rendered `buttonSpec.Dir` is now only the lookup key.**
   - Set: the `buttonSpec.Dir` field comment (updated), the file header of `buttonclick.go` (updated) and `reedengine/doc.go` (updated).
   - `buttonrow.go:1` and `:19` describe the rendered spec and `buttonDirOption` at render time, which stays true. Covered.
   - `reedengine/doc.go:85-95` is consistent with the code.

5. **The `button` verb now resolves a location from the told directory.**
   - Set: the `reedcli/button.go` `Long` text (updated), `reedcli/doc.go` (updated), the guard comment in `cli.go` (updated to "from its cwd"), and the `cli.go` pre-run guard.
   - The guard still skips the pre-run for `button`, so no behaviour change follows. Covered.

6. **PATTERN fit.**
   - `PATTERN-cwd-resolution`: `reedengine` receives the reader as a function and does not import `lymcwd`. `reedcli` already imports `lymcwd`.
   - The pattern file permits `ResolveWorktree` and `AnchorPath`, and says only `lymcwd` reads the marker. Compliant.
   - No `PATTERN.md` or `pattern/` entry is declared as changed, and the diff changes none. Nothing to land.
   - The commit has no `Pattern-Departure:` trailer, and none is needed.

7. **Docs and stencils outside the two packages.**
   - A search of `*.md` and `*.yaml` (stencils, specs, contracts) found no text describing the button's run directory. Nothing is left stale.
   - `internal/reedengine/template_posix.yaml` and `template_windows.yaml` hold the button commands and need no change.

8. **Second candidate cause (`hubgeom/buttons.go:73,81`).**
   - It was left unaddressed. The answer key says no code path producing it was found, so this is not a finding in my lens.
   - The `hubgeom` anchors come from `NamedWorktree(...).AnchorPath()` at render time. The click path now re-reads the anchor of whichever worktree holds that directory.

**Findings:** none.

**Tool calls (2):**
1. A grep of `*.md`, `*.yaml` and `*.go` for button and run-directory text, to find docs, stencils and specs the diff leaves stale. My context held none of them.
2. A grep of `buttonrow.go`, `buttons.go` and `hubgeom/doc.go` for `Dir` text, plus all `composeConfirmedButtonLine` callers. My context lacked the callers and the render-side docs.
