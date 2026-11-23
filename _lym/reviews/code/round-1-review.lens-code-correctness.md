```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each branch the diff changes in `internal/driftengine/verdict.go`, running one concrete input through each.

1. `renderVerdict`, baseline line (verdict.go:57-59)
   - Case: scope full, `LastCheck.Head="head2"`, `baseHead="code-head-0"`. The line renders "- Baseline: captured at `code-head-0`, the base of this branch." with the same format verb and argument as before, so only the literal changed. Correct.
   - Boundary: `baseHead=""`. The guard at :57 still skips the line, so no baseline line is emitted. Correct.
   - Boundary: `scope.Deferred` or no last check. The baseline branch is never reached, same as before. Correct.
2. `scopeText`, off with a source (verdict.go:99-100)
   - Case: `Scope=ScopeOff`, `ScopeSource="drift.scope"`. Returns "off, set by `drift.scope`; this change is not regression-checked." The single `%s` still matches its single argument. Correct.
3. `scopeText`, off with no source (verdict.go:102)
   - Case: `Scope=ScopeOff`, `ScopeSource=""`. Returns "off; this change is not regression-checked.", a constant string. Correct.
4. `scopeText`, not off (verdict.go:96-97)
   - Unchanged, returns "full.". Correct.
5. Error paths: the diff adds or changes none. `Verdict`'s error handling at verdict.go:23-33 is untouched, and the changed functions are pure string renderers with no calls that can fail.
6. Downstream parse risk: no code outside the tests matches on the changed literals, so the new wording cannot break a parser. The grep for "Baseline: captured" and "not regression-checked" found hits only in verdict.go, verdict_test.go and a comment at records.go:18. Correct.

Findings: none.

Tool calls: none. Everything I needed was already in context: verdict.go read whole, the diff, and the grep results for the changed literals.
