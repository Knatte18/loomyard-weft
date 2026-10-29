```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what each changed signature or output forces outside the diff's own lines.

**Per change, the set found:**

1. **`BoundaryBlock(event, text)` signature**
   - Implementers: only `claudeengine.Claude`, `capabilities.go:18` and `:72`. The interface at `engine.go:243` is updated. Covered.
   - Callers: `hookread.go:116`, `cli_test.go:161`, `settings_test.go:206`, `shuttleengines_test.go:344-345`. All updated, and a repo-wide grep shows no stale call. Covered.
   - `shuttleengines.ToolBoundaryRenderer` (`shuttleengines.go:254`) only returns the interface, so it needs no change. Covered.

2. **Stop block JSON shape**
   - Docs: `claudeengine/doc.go:134-137` and `creelcli/doc.go` are updated. `docs/overview.md:479` says "in a Stop block" without a field-level claim, so it stays accurate. Covered.
   - Tests: `TestHookContext` and `cli_test.go` use `HookEventStop` for the Stop rows and `HookEventToolBoundary` for the others. Covered.
   - `hookcmd_integration_test.go:340` uses a stub block with a fixed `mail` reason. The stub only checks the merged command's exit-code handling, not the real shape, so it needs no change. Covered.
   - `smoke_llm_test.go` (`TestSmokeCreelHookDelivery`) names no `reason` or `additionalContext` in the lines I searched. It is tier3, so I did not read it further. Covered.
   - `docs/research/*` reports are dated measurement records, so they are left alone. Covered.

3. **Tool-boundary block**: output unchanged and still pinned by `ToolBoundaryBlock`. `doc.go:133` keeps its old JSON text. Covered.

4. **Declared PATTERN changes**: the directive declares none, and none are needed. `hook-read` stays engine-neutral and the Claude specifics stay under `claudeengine`, which fits `PATTERN-shuttle-provider-seam`. Covered.

5. **Commit messages and departures**
   - There is no `Pattern-Departure:` trailer and none is needed.
   - The commit messages match the diff. The 10-of-900 failure figure is a benchmark claim in the commit body that I could not check from the diff.

6. **CLAUDE.md rules**
   - Docs landed in the same commits.
   - Semantic line breaks hold in the `.md` files the diff touches.
   - `doc.go:135-136` uses a `;` break, which is fine.

**Findings:** none from this lens.

**Tool calls (3 of 6):**
1. Grep for block-shape, renderer and Stop-wording mentions across `.go`, `.md`, `.yaml` and `.txt`. My context did not list every reader or asserter of the block.
2. Read `hookcmd_integration_test.go:325-350` and grepped `hook-read` mentions across the repo. I needed to know whether the stub block asserts the shape, and which files mention `hook-read`.
3. Grepped `docs/overview.md`, `hookread_test.go`, `smoke_llm_test.go`, `cli.go` and `doc.go` for `reason` and `additionalContext` mentions. I was checking for stale descriptions of the Stop block.
