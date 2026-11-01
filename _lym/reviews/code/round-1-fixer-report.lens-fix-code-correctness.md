```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the F1 test row and the F2 doc edit.

No findings.

- **F1 row `PrefixLongerThanForm`, run on `refuseRaw = ["go build ./internal/x extra"]`:**
  - **`go build ./internal/x`:** this form has 3 words and the prefix has 4. `len(prefixWords) <= len(words)` is false, so `refusedRaw` returns false and the form stays allowed.
  - **`go vet ./internal/x`:** the same: 4 <= 3 is false, so it stays allowed.
  - **Result:** `compileCheckSentence` returns the two-form sentence, which matches the row's `want` string exactly.
  - **Notice check:** `row.Notice == row.Steer+"."` holds.
- **Defect check:** `strings.Fields` allocates `make([]string, n)`, so a 3-word form has cap 3. With the guard removed, `words[:4]` panics with a slice-bounds error inside `Rows`, and the parallel subtest fails. The row therefore fails with the defect in place.
- **Callers:** the diff changes no signature or production code, only a test row and the `docs/overview.md` sentence. The gate run passed all of `./internal/shellguard`.
- **F2:** the new overview wording matches `compileCheckSentence`. The check is offered only while no prefix refuses it, and the sentence is empty when every check is refused.

No further tool calls were made.
