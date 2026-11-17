```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I traced each function and branch the diff adds or changes, step by step, with a concrete input.

**Per function or branch: the case I ran, and the result**

- `PlanMutants`, function-body site whose start lies in no block: `blockAt` returns false, so `blocks` is nil and the mutant is skipped. Correct.
- `PlanMutants`, package-level variable site (`holdFreeProducers` members): `readers[holdFreeProducers]` holds `RunHolds`'s return block, the blocks are sorted and compacted, and the mutant is planned when that block is a target. Correct.
- `PlanMutants`, variable read only inside another package-level variable's initializer (`var b = append(a, "y")`): `a` has no reader block, so `a`'s literals get no mutant. This errs toward fewer mutants and is not documented (finding 2).
- `parsePackageFiles`, file holding only `var` declarations: `MatchFile` picks it up and it is parsed. Correct. Two side issues:
  - `MatchFile` uses `build.Default`, not the tags the gate's builds pass (finding 3).
  - Dropping `parser.SkipObjectResolution` turns the deprecated object resolution back on (finding 4).
- `parsePackageFiles`, failure of `ReadDir`, `ReadFile`, `MatchFile` or `ParseFile`: the error is returned, and `planMutants` wraps it as "plan the mutants". Correct.
- `scanPackageDecls`:
  - `const ( a = iota; b )`: no `BasicLit`, or the value count differs from the name count, so neither is mapped. Correct.
  - `const n uint8 = 3`: maps to `lymMutantInt`. Correct.
  - `var _ = "x"`: skipped. Correct.
  - Embedded file missing: the error names the variable. Correct.
  - `template.yaml`: non-blank lines 1 to 7 are recorded. Correct.
- `embeddedFile`, `//go:embed template.yaml` on an ungrouped `var configTemplate string`: `value.Doc` is nil, so `gen.Doc` is used and `template.yaml` is returned. Correct. A named string type or a glob pattern returns false. Correct.
- `localNames`, `Shadow(text string)` with `suffix := text`: both names are locals, so the fixture's shadow row holds no mutant. Correct.
- `markFixed` and `variableReads`:
  - `Task{Slug: x}`: the key ident is fixed. Correct.
  - `x.Sel`: `Sel` is fixed. Correct.
  - `map[string]bool{"Worktree-Create": true}`: the key literal is fixed although map keys need not be constant (finding 1).
- `fileSites`, var-decl branch:
  - `var a, b = 1, "s"`: each value gets its own name. Correct.
  - `var a, b = f()`: both names attach to `f()`'s literals. Correct.
  - A `FuncLit` in a var value: skipped. Correct.
- `fileSites`, `*ast.Ident` branch:
  - `return raddleStoreDirName`: becomes `lymMutantString(raddleStoreDirName, id)`. Correct.
  - Untyped constant in a named-type or float context: does not compile and is dropped. The build uses `-gcflags=&lt;pkg&gt;=-e`, so there is no 10-error cap, and the 5 build rounds suffice. Correct, and the summary states it.
  - Embedded var on an assignment's left side: does not compile and is dropped. Correct.
- `addClauses`:
  - `a &amp;&amp; b || c`: the `||` node adds `(a&amp;&amp;b)` and `c` with `DropOr`, and the `&amp;&amp;` node adds `a` and `b` with `DropAnd`. Correct.
  - `a &amp;&amp; b &amp;&amp; c`: the outer node skips the same-operator left operand, giving 3 clauses. Correct.
  - `p != nil &amp;&amp; p.x`: deleting the first clause panics, which counts as a kill. That is the intended meaning.
- `lymMutantDropAnd` and `lymMutantDropOr` on a named bool or untyped bool operand: `T` is inferred. Correct.
- `lymMutantLine`, input `"one\n\n  two\n"` with line 3: returns `"one\n\n  ~two\n"`. Correct. The plan reads the same file the build embeds, so the index cannot go out of range.
- `applyMutants`:
  - Two mutants on the `s == names[0]` span: the delete clause is added first (outer `||` visit), so it gets the lower rank and opens outer and closes last. This matches the test.
  - Dropped statement ending at the same offset as its last clause: the shorter closes first. Correct.
- `Mutant.opening` and `Mutant.closing`: empty helper, helper without args, and helper with args all render as pinned. Correct.
- `Mutant.InBlocks`, nil `BlockSet`: false. Correct.
- gatecli `run` and `runMutants`: `InBlocks` replaces single-block membership for candidates and covering tests. Correct.
- Done-when trace against the boardengine code (no full run):
  - `TestRunHolds`: the "Worktree-Create" literal mutant breaks the `BeamAtWorktreeCreate` row.
  - `TestRunLock_BeamKindAndBeamChange`: only the Beam-clause deletion is unique, because the `KindChange` row also changes Recipe and Beam. One unique kill still refutes the candidate.
  - `TestRunLock_SketchExemptionIsTheWholeNetChange`: deleting `now.Status == nil` (`sketch.go:83`) is killed only by `SketchThatKeepsItsRunStatusIsRefused`.
  - `TestReadRaddleStore`: the `raddleStoreDirName` and `raddleStoreFileName` uses at `raddlestore.go:150` change the exact command.
  - `TestConfigTemplate_ResolvesToDefaults`: the `~design_prefix`, `~bug`, `~enhancement` and `~labels` line mutants.
  - All five plausibly get a unique kill.

**Findings**

1. MEDIUM, design. `internal/testcover/mutant.go:482-483` (`markFixed`'s `*ast.KeyValueExpr` case).
   - Every composite-literal key is held fixed, map-literal keys included, though only array, slice and struct keys must be constant.
   - So a data table kept in a map's keys (`map[string]bool{"Finalize": true}`) gets no literal mutant.
   - The board entry names "map entry" among the missed data. The summary's "reaches … the entries of a map literal" holds for values only, and the doc's "since it must be constant" (`mutant.go:211`) is false for map keys.
   - Evidence: `mutant.go:482-483`, `mutant.go:211`; `_lym/landing/summary.md` "Package-level variable literals".
2. LOW, design. `internal/testcover/mutant.go:272-281` and `:505-525`.
   - Only function bodies count as reads of a package-level variable.
   - A variable whose only reader is another package-level variable's initializer (`var all = append(base, "x")`) gets an empty block set, so its literals are never planned.
   - Neither the doc nor the summary states this.
   - Evidence: `variableReads` inspects only `FuncDecl` bodies (`mutant.go:510-513`).
3. LOW, design. `internal/testcover/mutant.go:329`.
   - `build.Default.MatchFile` picks the extra files by the lym process's own context, not by the build tags the gate's builds use (`appendTags`, `runner.go:374`).
   - A tag-gated file's package-level constants and variables can be missed or wrongly included.
   - Evidence: `mutant.go:329`; `internal/testcover/runner.go:374-378`.
4. NIT, design. `internal/testcover/mutant.go:345`.
   - `parser.ParseComments` replaces `parser.SkipObjectResolution` instead of joining it.
   - Every file now pays for the deprecated object resolution, which nothing reads; `parser.ParseComments|parser.SkipObjectResolution` keeps the old cost.
   - Evidence: `mutant.go:345` against the old line in the diff, `parser.SkipObjectResolution`.

Tool calls:
1. Grepped `mutantBuildRounds`, `endsAsSketch` and `BuildMutantBinary`, to judge build-round exhaustion and the Sketch candidate's clause, which the manifest did not hold.
2. Grepped `BuildMutantBinary`'s flags (`-gcflags=-e`), to settle whether the compiler's 10-error cap could exhaust the 5 rounds.
