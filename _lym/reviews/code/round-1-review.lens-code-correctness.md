```yaml
lens: code-correctness
outcome: reported
```

**Per function / branch — case run — correct?**

- **`closing`/`opening` for `stepHelper`** (`mutant.go:166-187`): `s[0] == '0'`, id=5 → `('0' + 1>>lymMutantStep(5))`. `>>` binds tighter than `+`, so `'0' + (1>>step)`; off: step=1 → `1>>1 = 0` → `'0'`; on: step=0 → `1>>0 = 1` → `'1'`. Non-constant shift rule: the 1 takes the type `'0'+1` would have in the comparison with a byte → byte. Nested inside a comparison mutant: inner closing first → `…('0' + 1>>lymMutantStep(14)), 13)` (matches test row `mutant_test.go:187`). Correct.
- **Step rewrite in Duration context** (`5 * time.Second`): 1 becomes Duration (integer) → shifts fine. Correct.
- **Step rewrite in float context** (`5 * float64(x)`, `1.5 + 2`): 1 becomes float64 → compiler refuses "shifted operand 1 (type float64)", on the literal's own line (closing text is on the literal's line) → line mode drops it. Old generic helper also failed here; no regression. Correct.
- **Step rewrite in constant-only contexts:** array length, composite non-string key, const decl (package and function-body), field tag, selected name are all `markFixed` before the literal is visited. `case 'a':` labels stay legal as non-constant (expression switch allows it; only the duplicate check is lost). Correct.
- **Step rewrite as shift count** (`x >> 3` → `x >> (3 + 1>>f())`): untyped non-constant count converts to uint. Correct.
- **Typed package constant** (`const N byte = 5`, use → `(N + 1>>f())`): 1 becomes byte. Correct.
- **`lymMutantStep` off/on values:** 1 off / 0 on → adds 0/1. Correct.
- **`lymMutantLine`:** `"name: lym\n"` → sep 4, value `"lym\n"`, at=6 → `name: ~lym`. `"note = x\n"` → `=` at 5 → `note = ~x`. `"list:\n"` → no `": "` (colon then `\n`), no `=` → indent → `~list:`. `"  plain\n"` → at=2. `"key: \n"` → value trims empty → indent fallback. `"a=b: c"` → earliest sep (`=` at 1) wins. Last line without `\n` fine. Correct.
- **`UnbuildableMutants`** (`mutant.go:857-882`): non-`MutantBuildError` → nil → `buildMutants` returns err. `MutantBuildError` is only built with `len(lines) > 0` (`runner.go:369`), so the every-mutant-left fallback always has a line to match. Line mode `fix.go:4` → 1,2. `fix.go:1` unspanned → file fallback, all of `fix.go`. `wholeFiles` → file. `other.go` with 2 dropped → 1,3,4. All dropped → nil. nil `dropped` map reads safely. Correct.
- **`buildMutants` loop** (`redundantmutants.go:397-420`): each round drops ≥1 not-yet-dropped mutant → terminates. All dropped and still failing (e.g. helper file refused) → `UnbuildableMutants` nil → returns err. Non-compile failure (timeout, no parseable line) → returns err. Correct, bar the doc off-by-one below.
- **`compileErrorLine`:** `./overlay-1/x.go:4:9: ` → x.go:4; `/abs/overlay-3/z.go:62: invalid` → z.go:62 (column absent); `# m/p` header not matched. Correct.
- **`ScanTestFuncs` with `build.Default.MatchFile`:** `d_test.go` `//go:build !<GOOS>` → skipped; `foo_linux_test.go` on Linux → kept; unparsable body (`func {`): MatchFile reads only the header, then ParseFile fails naming the file. Map builds pass no tags (`testmappackage.go:203` `BuildBinary(..., "", ...)`), so the tag-excluded set matches the binary. Correct.
- **`redundancy.go:585` `judgeRuns`:** a mapped test whose file is now excluded drops from `current` → no longer holds; consistent (it cannot run on this host). Correct.
- **Fixture `Timeout` → `time.Duration(5 * float64(time.Second))`:** the mutated 5 is in a float context → still refused on its line; the fixture's stated purpose holds. Correct.
- **`TestMutantBinary_KeepsTheTypeOfAByteContextLiteral` expectations:** 4 mutants (comparison, index 0, `'0'`, the 2); each makes `TestTyped` fail (`"01"[1]='1'`; `'0'`→`'1'`; negated comparison; `4/3=1`). Correct.

**Findings**

1. **LOW · consistency** · `internal/gatecli/redundantmutants.go:35-36` — The `mutantBuildRounds` doc says it "is the number of builds … that drop only the mutants of refused lines", but the code passes `round >= mutantBuildRounds` with `round` starting at 1, so only builds 1–4 drop by line and build 5 already drops whole files. Evidence: constant doc `redundantmutants.go:35-37` vs call `redundantmutants.go:410` (`plan.UnbuildableMutants(err, dropped, round >= mutantBuildRounds)`, loop `for round := 1`). The gatecli doc line ("From build `mutantBuildRounds` on") agrees with the code and contradicts the constant's doc.
2. **LOW · design** · `internal/testcover/mutant.go:46-48` / `lymMutantLine` (`mutant.go:109-130`) — Marking the value instead of the key avoids an unknown-key refusal, but a strictly typed loader still refuses a marked non-string value (`port: 8080` → `port: ~8080`, `enabled: true` → `enabled: ~true`), so every loading test kills that mutant again, the false-cover pattern the change set out to remove. Evidence: value marked unconditionally regardless of type (`mutant.go:122-127`); doc claim at `mutant.go:46-48` and `doc.go:22-23` speaks only of unknown keys, so accurate, but the remaining case is unhandled/unstated.

No tool calls made — all traced from inherited context.
