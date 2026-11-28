```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens, tracing each change in the diff to the board entry's goal points and to the record's `Added after Discussion` entries.

**Per change**

- **C1: template jacquard blocks and default** (`internal/runprofile/template.yaml`). The `speed`, `light`, `solid` and `wide` profiles gain `jacquard` blocks, which are their tapestry block minus `loom.discussion`. `defaults.jacquard: solid` is added, and the header comment gains a line describing the jacquard block. Traces to Goal "Takes" (first clause) and "Exports". The header line is a doc edit forced by C1.
- **C2: one-time reinterpretation** (`internal/runprofile/reinterpret.go`). Adds `ReinterpretJacquardBlocks`, `reinterpretJacquardDefault`, `jacquardSettings`, the `ownMapping` and `setMappingValue` helpers, and the `jacquardRecipe`, `loomModule`, `discussionArea` and `JacquardBlocksReinterpretationID` constants. It handles aliases and merge keys. Traces to Goal "Takes": the reinterpretation in `internal/runprofile` beside `ReinterpretArrasDefault`, block carry, set or drop of `defaults.jacquard`, and the literal block key `jacquard`. It also traces to "Delivers".
- **C3: shared `encodeDocument`** (`reinterpret.go`, arras function). The arras function's encoder moves into a shared helper, with no change in behavior. Forced by C2, whose rewrite needs the same encoder.
- **C4: registration after the arras entry** (`internal/configreg/configreg.go:171`). Traces to Goal "Takes": "registered in internal/configreg's runprofile entry after the arras one".
- **C5: package doc paragraph** (`internal/runprofile/doc.go`). Forced by C1 and C2, since the package doc describes the profiles and their reinterpretations.
- **C6: tests.** These are a forced test edit for C1, C2 and C4:
  - `reinterpret_test.go`, new;
  - the `TestLoad` and `TestTemplateLadder` rows in `runprofile_test.go`;
  - the order test in `configreg_test.go`;
  - the `boardcli/verbs_test.go` defaults expectation, which C1's new default makes stale.
- **C7: commit 3d4cf8bcb, comment line breaks.** Forced by C1 and C5 (prose layout of their comments).

**Per decision**

- **Goal "Takes":** built by C1, C2 and C4. `internal/hubreconcile/walk.go` is left unchanged, as the entry says.
- **Goal "Delivers":** built by C1 and C2. Nothing else is required, since raddle's check reads the blocks.
- **Goal "Exports":** built by C1. The four profiles hold jacquard blocks, and `defaults.jacquard` is `solid` in the template.
- **Added "Jacquard stays one recipe of every row after Discussion…":** asks this part for no code. No change adds a plan or build recipe, which is consistent.
- **Added "The jacq nickname is written by the run's parent after landing" (arbiter) and "Nickname jacq written after landing" (parent):** both ask for no code in the run. No file in the diff writes `heddle.json` or a nickname line, which is consistent.
- **Added "Jacquard gains no motif-run clear":** asks this part for no code. No change touches batten or motif clears, which is consistent.

Findings: none. Every change traces to a goal point or is forced by a traced change, and every goal point is built.

Tool calls: none.
