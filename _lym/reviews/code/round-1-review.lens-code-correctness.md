```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the advisor diff. Per function or branch, the case I ran:

- `run.go` flag `--advisor` into `Spec.Advisor`: the flag set to `opus` gives `Advisor:"opus"`, and the flag absent gives `""`. Correct. The value is passed through unvalidated, as `Spec` documents.
- `buildSettings` with `advisorModel==""`: `omitempty` drops the key. The document differs from today's only by an absent key, so it is byte-identical. Correct.
- `buildSettings` with `"opus"`: the key is emitted after `hooks` and before the embedded `sessionScope` keys. Correct. Only a key-name collision would break it, and `sessionScope` has no `advisorModel`.
- `Prepare` in claudeengine passes `spec.Advisor` to `buildSettings`, and the resume line reuses the same `settingsPath`, so a revived session keeps its advisor. Correct.
- `validateSpec` in codexengine with `Advisor:"opus"`: it returns the error after only `loadConfigOrRefuse`, a read. Nothing is written before `readyHome`, so there is no partial state. The error names the way forward. A resumed session is checked first, so a spec with both gets the resume message. Correct.
- `validateSpec` with `Advisor:""`: unchanged. Correct.
- Opened sessions (`buildOpenSettings`, `buildOpenCmd`) take no advisor. The decision record scopes the option to `lym shuttle run`, so that is correct.
- Empty or whitespace `--advisor " "`: a non-empty string reaches the settings file as `" "`. This is the boundary and trust-boundary pass-through the spec states, and Claude Code validates it, so I count it correct.

No findings.

Tool calls:
1. `sed -n 98,125p internal/shuttleengine/codexengine/codexengine.go` plus `grep -n settingsPath internal/shuttleengine/claudeengine/command.go`: I needed the order of `validateSpec` against the first write in codex `Prepare`, and the settings reuse in the resume line. My context had only the diff hunk.
