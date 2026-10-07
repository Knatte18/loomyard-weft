# Fixer report — webster round 1

All four findings of `round-1-review.md` are fixed, one commit each.
`go test ./...` passes and `lyx loom lint-comments --base <merge-base>` reports `ok`.

## Fixed

- **F4 (NIT)**, `fa6cae8f8`: split the `resolvePromptCacheTTL` doc sentence that chained two "so" clauses into two sentences, one per line.
- **F1 (LOW)**, `98c250548`: `resolvePromptCacheTTL` returns the default for an empty role before it looks the map up, so a map key of empty string no longer overrides it.
  The new row `ttl_empty_role_ignores_empty_map_key` in `TestPrepare_ThreadsSpecIntoLaunchCmds` failed before the guard and passes after it.
  It extends the existing table, so no new test function and no `test-budget.yaml` change.
- **F3 (LOW)**, `3af262d22`: the `ttl_default_wrong_case` and `ttl_default_bare_number` rows assert `claude_prompt_cache_ttl "1H"` and `claude_prompt_cache_ttl "5"`, so they pin the key as well as the value.
- **F2 (LOW)**, `199c68e39`: every non-TTL row of `TestPrepare_RejectedBeforeArtifacts` sets `cfg: templateConfig(t)` explicitly, the `reflect.DeepEqual` zero-Config fallback and the `reflect` import are gone, and the `ttl_default_empty` row's `shuttleengine.Config{}` now means exactly that.

## Deferred

Nothing.
