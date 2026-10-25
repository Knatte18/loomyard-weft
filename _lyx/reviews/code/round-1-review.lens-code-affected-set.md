```yaml
lens: code-affected-set
outcome: reported
```

Task: list what the diff's changes force outside the lines they touch (callers, tests, docs, fixtures) and report the gaps.

**Set per change**

- **`readRecordedAnchor(hub, worktreeName)` signature:** the callers are `NamedWorktree` (`internal/lyxcwd/lyxcwd.go:119`) and `resolveCore` (`:151`). Covered, with no other caller in `internal`.
- **`NamedWorktree` / Location literals (D3 scan):**
  - Three non-test files mention `Location{`: `internal/battencli/overview.go:300`, `internal/tendercli/deps.go:55` and `internal/orchcli/live.go:28`.
  - All three are map or slice types, not composite literals of a Location.
  - The scan fixtures accept those shapes (`enforcement_test.go:957-958`).
  - Covered.
- **`PATTERN-cwd-resolution` overview line (declared change):**
  - The `PATTERN.md` line is 357 characters, under `MaxEntryLineChars` = 360 (`internal/pattern/check.go:27`).
  - The `(test)` marker is present.
  - Covered.
- **`pattern/PATTERN-cwd-resolution.md` (import set, `NamedWorktree` line):** covered.
- **`pattern/PATTERN-fabric-vocabulary.md` (`weftname` import carve-out):** updated to name `internal/lyxcwd`. Covered.
- **`docs/shared-libs/lyxcwd.md` (read order):** updated at lines 33 and 54-55. Covered.
- **`docs/overview.md` (line 178):** it still describes junctions at "the repo's lyx-anchor". I read only that line, not the module-table or execution-stack context.
- **Everything else:**
  - I did not trace the engine, CLI, dobby, adopt and fabric-anchor verb changes, or the fabric/beam CLI help.
  - That includes the `fabricengine/doc.go`, `fabriccli`/`beamcli` doc and CLI help changes the record requires.
  - I made no judgment on them.

**Findings:** none from the lens within the budget.

Tool calls (2 of 6):
1. A grep for `readRecordedAnchor` callers, the `MaxEntryLineChars` definition, the PATTERN line length, the anchor mentions in the docs, and non-test `Location{` sites. My context lacked all of it.
2. A grep of the three `Location{` hits plus the scan test fixtures, to confirm they are not composite literals.
