```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix set for agreement with its surroundings (fix-consistency lens).

**Findings**

1. **LOW** — `internal/shuttleengine/claudeengine/subagents.go:37-47`, `subagentEffortEnv`. This is the twin left unchanged after the F5 change.
   - Its doc says it "refuses an effort Claude Code does not accept", and it returns an error wrapped as `agents.yaml effort of role %q`.
   - `agentconfig.Load` now refuses every effort outside `agentconfig.Efforts()`, and `Config` is built only by `Load` (its fields are private).
   - So the check, its error path and the caller's error handling can no longer be reached. The doc also names a refusal that now happens earlier, at load.
   - The new `effortVocabulary` is the single source, so nothing contradicts it. This is stale redundancy, not a wrong result.

2. **No finding** — `internal/agentconfig/template.yaml` comment. It now says "validated when the file loads", which matches `checkEfforts`. The only other mentions of the old wording, `command.go` and its test, still agree with the shared vocabulary.

3. **No finding** — `validateEffort` error text. It is built from the same list and gives the same string as before, so no existing assertion or doc drifts.

4. **No finding** — F6, `configcli.go`. The restored line sits at its original position, as the `a1568d23` diff shows. The `configcli_test.go` menu change belongs to commit `00a08d6d` and is unrelated to help text, so leaving it is correct. The review's "revert the matching test change" does not apply.

**Calls**
1. `grep` over `internal` and `docs` for effort-validation wording: my context lacked any search for docs or comments that describe the old "validated when a session starts" behaviour or the effort check's location.
