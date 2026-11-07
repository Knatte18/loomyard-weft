```yaml
lens: fix-affected-set
outcome: reported
```

Task: check the sets behind F1–F4 (callers of `foldObjects`, record-removal sites, test rows) under the fix-affected-set lens.

No findings.

- **F2 (callers of `foldObjects`):** the search found two call sites, `render.go:355` in `Show` and `settings.go:244` in `renderObjectTarget`. Both take the new four-value signature. `Show` ignores `invalid` and the warnings, as it did before.
- **F3 (record-removal sites):** the search for `recordPath` and `.rendered.json` found one removal helper, `removeRecord` at `settings.go:212`. Its two uses are `settings.go:286` and `settings.go:297`, which are the file-removed and file-written branches of the not-held case. Both come after the file change. No raw `os.Remove` of the record is left.
- **`dropStale`:** the only callers are its own recursion and `settings.go:276`. The latter now runs for every non-invalid render, including a nil `previous`, which `dropStale` handles (a nil map ranges over nothing).
- **F1 and F4:** these are test additions, not set-based findings, so I did not re-run a search for them.

I made 2 further tool calls:
1. `grep -rn` for `foldObjects(`, `recordPath` and `removeRecord(` across `internal`. The output was too long, so I discarded it.
2. `grep` narrowed to `internal/heddleengine` and `cmd` for `foldObjects(`, `removeRecord(`, `dropStale(`, `rendered.json` and `recordSuffix`. Neither the `grep` nor the post-edit call sites were in my context.
