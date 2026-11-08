```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set — what the diff forces outside its touched lines.

**Per change, the set found:**
- Hook pinned on every layout path: the only `select-layout` issuers are `spawn.go:214` and `ownwindow.go:303`, and both are followed by a pin. Covered.
- Window-size unpin vs group hook: `windowsize.go:127-128` unsets `window-resized` only on `target`, the shell window, so it cannot touch a group window's hook. Covered.
- `windowResizedHookName` is read only in `windowsize.go` and `ownwindow.go`. The `watchdog.go` comment is updated. Covered.
- `doc.go` statements on resize hooks: lines 341-343, 746-750 and 856 are updated. Line 1106 (the shell-window clear) and line 1117 stay correct. `attach.go:51` and `lifecycle.go:579` describe only the shell-window unpin and stay correct. Covered.
- Existing tests of the unpin (`windowsize_test.go`, `attach_test.go`, `lifecycle_unix_test.go`, `contract_integration_test.go`) all target the shell window or a scratch window and are unaffected. Covered.
- `PATTERN.md` and `pattern/`: the diff declares no change, and `pattern/PATTERN-reed-panes.md` does not mention hooks or layout. Nothing to land.
- Commit message and `summary.md`: the claims about Windows, the zoom skip, per-layout replacement and keeping `codexengine` unchanged match the code. There is no `Pattern-Departure:` trailer and none is needed.

**Findings:**
- **F-a** — LOW, `consistency`, `internal/reedengine/doc.go:341-343`. Line 341 says reed "keeps no resize hook" in the shell window. Line 343 now says "The only resize hook reed keeps is a group window's own". Line 343 sits in the shell-window paragraph and implies no other resize hook exists. Line 856 correctly lists the shell-window unpin as a hook-related step, so this is only a mild inconsistency. Fix: reword line 343 to "A group window's own resize hook, which issues that layout again, is the only one reed pins."

I found nothing else the diff leaves wrong.

**Tool calls:**
1. grep for resize-hook and `window-resized` mentions across `.go`, `.md`, `.tmpl` and `.yaml` outside `_lym` and `.lym`, to find asserters, docs and stencils. The manifest had only the changed files.
2. grep of `pattern/PATTERN-reed-panes.md` plus a diff stat of `PATTERN.md` and `pattern/`, to check the rule entry and for declared changes.
3. grep for all `select-layout` and layout-name sites in non-test Go, to confirm every layout path is pinned.
