```yaml
lens: fix-affected-set
outcome: reported
```

Task: re-run the F1 search (every site that states the quoted-shell-string wording) and check the fix covers all of them.

No findings.

The one finding is F1, and its premise is a set of sites that carry the overstated wording. The search over `.go`, `.md` and `.yaml` files outside `_lym/` and `.lym/` turned up exactly three sites for the `MigrateConfig` quoted-shell-string behavior:
- `internal/driftengine/doc.go:20` carries the corrected wording, "ends a quoted shell string".
- `internal/driftengine/diffmode.go:40` carries the corrected wording.
- `internal/driftengine/diffmode.go:30`, the `skipFileArgument` comment, says "the quote that closes an enclosing `sh -c '...'` string is never taken". That text is accurate and consistent with the new wording, so it needed no change.

The other matches (`refscanner`, `claudeengine`, `shellguard`, `shell`, `codestats`) concern different features.

I did not check `_lym/`, `.lym/` or non-text docs. Neither is in the F1 set.

Tool calls:
1. `grep -rniE` over `.go`, `.md` and `.yaml` files for "quoted shell", "inside a quoted" and "sh -c '...'". My context held only the three driftengine files, so I needed this to find any other site with the wording.
