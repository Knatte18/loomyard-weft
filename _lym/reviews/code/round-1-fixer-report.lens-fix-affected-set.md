```yaml
lens: fix-affected-set
outcome: reported
```

No findings. Each set-premise finding was re-searched and the fix covers every member I found.

- **F1 (embed scope, callers of `otherFileMiss` and `embedPatterns`):** the callers are `testselect.go:209` (own package, `true`), `:249` (linked, `false`) and `:412`. The cache key at `:427` and `:461` uses `embedScope`. No other caller exists.
- **F7 (`importAllowed`, `GoRoot`):** the single caller is `:260`. `NewSelector` and `SelectorConfig` have no user outside the package's own test, so no production caller leaves `GoRoot` unset. This is LOW and not a defect: the new `GoRoot` field is on the type, and a future caller has to set it, because an empty `GoRoot` makes every added standard-library import a miss.
- **F8 (comments naming other symbols):** no `ContentHash`, "as Dir" or "as a block key" text remains in `testselect.go` or `declcompare.go`. `testselect.go:348` is a code call, not a comment. `testselect.go:42` ("as its go.mod declares it") names a file, not a symbol.
- **F4 (D5 cases):** all the missing-row cases the review named have a row, with two exceptions.
  - A method named `max` now wants `MissMethodSet`, not "no miss": an added method is a method-set change. This is a deliberate own-change and the fixer should record it.
  - The recorded non-Go file removed has a row, but only with `inert` nil.
- **Docs:** `doc.go:63-65` still agrees with the behaviour.

Calls made:
1. Grep of non-test callers and `ModulePath` uses, to rerun the F1 and F7 caller searches.
2. Grep of `ContentHash` and "as …" phrases, to rerun the F8 search.
3. Grep of `GoRoot`, `SelectorConfig` and `NewSelector` outside the package, to find any caller left without the new field.
4. Read of `doc.go:50-70` and grep of `docs/overview.md`, to check the docs still agree with the standard-library and embed behaviour.
