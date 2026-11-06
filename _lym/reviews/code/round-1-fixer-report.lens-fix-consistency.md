```yaml
lens: fix-consistency
outcome: reported
```

No findings.

The change set agrees with its surroundings:
- No file still carries the old `otherwise parse JSON with jq` wording. Only `rows.go` and `settings-golden.json` carry the new wording.
- `codexSteerPythonDeny` and `steerPythonDeny` both derive from `pythonRoutes`, so the Codex steer picks up the F3 wording automatically.
- `noticePythonDeny` uses `pythonRefusal` only, so the notice fixtures in the golden file correctly stay unchanged.
- The F1 assertion's `HasPrefix` against `pythonRefusal` holds, because `pythonRoutes` begins with `pythonRefusal`.
- The `TestCheckTexts` doc comment now matches the code in the loop.
- The `TestConfigLong` doc comment matches its `wants` list again.

One call:
- A grep over the repo for the old wording, the new wording and the `pythonRefusal`/`pythonRoutes` references outside `rows.go`, because my context held no repo-wide view of twins.
