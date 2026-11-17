```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- **F2:** a search of `tools/retro`, `internal/retroengine` and `internal/retrocli` (Go, shell and jq) finds `_lym` or `.lym` in a path only in the `USAGE_DIR`, `SHED_DIR` and `REVIEWS_DIR` block of `tools/retro/retro.sh:24-26`. That block is the `tools/` script variable-block exception, so no test literal is left.
- **F1:** every `runs[...]` index in `Select` is covered. Line 187 is safe because the branch only runs when `len(runs) > MaxRuns`. Line 189 is bounded by `cut > 0`. Line 194 now has the `cut < len(runs)` guard. Lines 169 and 199 index a non-empty slice, and `cut >= 1` at line 199 because `runs[0]` always matches `firstLeftOut` when `cut` reaches 0. The new "every run in one second" row traces correctly: `cut` goes 1, then 0, then widens to 2, so the plan keeps `a` and `b`.
- **Unchecked:** none. F3, F4 and F5 are not set-premise findings.

Tool calls:
1. Grep for `_lym` and `.lym` literals across the three retro packages: my context lacked a search over files the fix did not touch.
2. Grep for `runs[` in `retro.go`: my context lacked a search for other indexing sites in the same function.
