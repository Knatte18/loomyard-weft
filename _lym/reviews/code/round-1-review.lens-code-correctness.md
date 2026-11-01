```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness of the diff's changed code (`compileCheckSentence`, `refusedRaw`, the python and raw-go texts).

**Per function or branch, with the case I ran:**

- **`compileCheckSentence`, `nil` refuse list.** Both forms are allowed, giving "A compile check names its packages, as in go build ./internal/x or go vet ./internal/x, never ./... or all." Correct.
- **`compileCheckSentence`, `["go test"]`.** The prefix words `[go test]` differ from `[go build ./internal/x]` at index 1, so both forms are allowed. Correct.
- **`compileCheckSentence`, `["go","go build"]`.** `go build` refuses the build form and only `go vet ./internal/x` is named. Correct.
- **`compileCheckSentence`, `["go"]`.** Both forms are refused and the sentence is `""`. The steer then reads "...go runs are refused here. Gate rule: ...", which is well formed, and the notice is steer plus ".". Correct.
- **`compileCheckSentence`, an empty or nil `refuseRaw` result.** `allowed` is only appended to, so the nil slice is never joined. Correct.
- **`refusedRaw`, a prefix longer than the form** (for example `go build ./internal/x ./y`). `len(prefixWords) <= len(words)` is false, so it is not refused. The real `refuseRawPattern` would not match `go build ./internal/x` either, since it needs the extra word. Correct.
- **`refusedRaw`, a prefix that ends mid-word** (for example `go build ./internal`). The words differ, so it is not refused. The real pattern's word-end class `[^A-Za-z0-9_./-]` excludes `/`, so `./internal` before `/x` does not match. Correct and consistent.
- **`refusedRaw`, a prefix with flags or `./...`** (for example `go build -v`, `go build ./...`). It is not a word prefix of the offered form, and the real pattern does not match the offered form either. Correct.
- **`refusedRaw`, a blank prefix.** `strings.Fields` gives no words, `slices.Equal` is true, and everything counts as refused, so no compile check is offered. The real pattern with an empty alternative also matches any command position. The two agree, so it fails safe. I did not check whether the gate config loader rejects a blank prefix, so I mark it not applicable.
- **`go build ./internal/x` and `go vet ./internal/x` against `rawGoCommandPattern`.** The argument word `./internal/x` holds no `...` and is not `all`, so neither command is denied. The new `rows_integration_test.go` rows pin this. Correct.
- **`Rows` wiring.** The sentence has a leading space and `"here."` precedes it, so spacing is right with or without the sentence. The notice (`rawGoSteer + "."`) stays equal to the steer plus a full stop.
- **Init-time forbidden-character checks.** The new sentence has no `'`, `"`, backslash or newline, and neither does `pythonRefusal`. The sample-refuse-list check in `settings.go` at line 466 renders through the same `Rows`. Correct.
- **Python steer and notice.**
  - `python --version`, `python3 --version` and `python3.11 --version` all match `pythonCommandPattern` in command position, so the claim that a version check is refused holds.
  - The edit rule is still in the steer through `editToolsRule`, and the notice still omits it, so the edit-rule count stays at 1.
  - The gate-rule count stays at 1, because the python notice does not contain "Gate rule".
  - When `deny_python` is off the row is not installed, so the text "refused in every lym session" is never shown. Correct.
- **`TestRows_RawGoSteerNamesOnlyUnrefusedCompileChecks`.** The four rows give correct results. One weakness: if `Rows` ever returned no `raw-go` row, the loop would find nothing and the test would pass without asserting. This is a weak guard only, not a defect.

**Findings:** none.

I made no tool calls; the context already held everything I needed.
