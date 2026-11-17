```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: no findings.

Checked each finding whose premise is a set:
- **F1 (docs describing the nudge and deny as Claude-only):** other `hook-batching` and `batching_nudge` mentions are only in `docs/research` measurement reports, which are exempt. `request_user_input` appears in docs only in `docs/overview.md`.
- **F3 (`file://` URL handling):** `localPath` is the only `file://` parser in non-test Go code. The `url.Parse` in `githubclient.go` is unrelated.
- **F7 (`CountLinesUpTo` callers):** the two non-test callers, `claudeengine/batchingnudge.go:497` and `codexengine/batchingnudge.go:539`, only compare the result against `SliceLines`, so clamping the result to `limit` keeps them correct.

Left unchecked: F2 (a keep-marker set) and F4 through F9 (test-coverage rows). They are not site or caller sets I could re-search.

Calls made:
1. Grepped `docs`, `README.md` and `contracts` for `hook-batching` and `batching_nudge`, because my context had no search of the docs outside `overview.md` line 705.
2. Grepped Go sources under `internal` for `url.Parse` and `file://`, to find other `file://` parsers.
3. Grepped `docs` and `internal` for `request_user_input`, and `internal` for `CountLinesUpTo`, to find the other docs mentions and the callers.
