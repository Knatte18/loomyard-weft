```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

- **Numbered orch role, `ly:bm:orch-2` upserting `{"slug":"member","profile":"speed"}`:** `Parse` yields slug `bm` and role `orch-2`.
  - `MatchesRole("orch-2","orch")` is true, so the guard does not refuse on role.
  - `project` falls back to the stored `"bm"`, and `BeamUnder("bm")` returns `"bm"`, which equals the slug. The upsert passes.
  - With `MatchesRole` replaced by `==`, the guard returns the refusal and `runJSON(t, 0, …)` fails on exit code 1, so the row fails with the defect in place.
- **Non-string project, `ly:bm:orch` with `{"slug":"member","project":5,"profile":"docs"}`:** `value.(string)` fails, so `project` becomes `""`.
  - `stored.Project` is `"bm"`, which differs from `""`, so `links` is `["", "bm"]`.
  - `BeamUnder("")` returns `""`, which differs from `bm`, so the guard refuses with the message the row asserts.
  - If a non-string value were let through and `project` stayed the stored `"bm"`, the guard would admit it and the row would fail on the expected exit 1.
  - The row asserts the strand name and "the orch of the entry's beam" in the error. That ties the refusal to the guard rather than to some other validation error.
- **Callers and state:** No signature changed, since the fix only touches the test file. The `t.Setenv` to `orch-2` is overridden by each refusal subtest's own `t.Setenv`. The earlier assertions on `new` are unaffected. The `member` profile change to `speed` does not feed any later row.
- **Gate:** `lym gate test ./internal/boardcli` passed.

Further tool calls:
1. Grepped `agentname.Parse` and `MatchesRole` in `internal/agentname/agentname.go`, because my context did not show how a numbered role parses or matches.
2. Grepped `BeamUnder` in `internal/boardengine/parentbeam.go`, because my context did not show its behaviour for an empty or beam-slug project.
