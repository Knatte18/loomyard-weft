```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: for each change, the set it forces outside the diff's lines, and any member left wrong.

**Per change, the set I found**
- `run --effort` removed: no other caller, doc or stencil passes `lym shuttle run --effort`. The remaining `--effort` hits are the claude launch line and `llm-model-spec.md:103`, which is claude's own flag. Covered.
- `run --model` now takes a model spec: the only non-test callers are `tools/sandbox/SANDBOX-SHUTTLE-SUITE.md:172` and `:197`, both `--model haiku`. `haiku` is a built-in registry alias (`internal/modelspec/registry.go:20`), so both still resolve. Covered.
- `docs/overview.md` shuttle paragraph: updated for the model spec, the codex `open` and the `model_not_found` end. Covered.
- `SessionSignal.ModelRejected` and the `retryAPIError` change: `shuttleengine/doc.go:121-123`, `spec.go:80`, `claudeengine/doc.go` and the fake engine in `fakes_test.go` are updated. Two statements that the `api-error` outcome needs `APIErrorRetries` were left as they were (see the finding below). Not fully covered.
- `OpenLaunch` and the `SessionOpener` assertion: `shuttleengine/open.go:70` and its test `open_test.go:53` still cover the refusal for an engine without the capability. `codexengine/doc.go` and `command.go` are updated. Covered.
- `.codex` check wording: no remaining "committed Codex" or "assist model key" text anywhere outside `_lym`, `.lym` and the research docs. Covered.
- PATTERN entries and background files: `PATTERN-shuttle-provider-seam` is respected. The provider-neutral `ModelRejected` field is set only in claudeengine, and the engine name reaches the CLI only through the registry. No declared PATTERN changes are listed in my directive. No `Pattern-Departure:` trailers appear in the commit messages. Covered.

**Findings**

1. **LOW, consistency.** `internal/shuttleengine/doc.go:75` says a run ends as "api-error (a spec with APIErrorRetries only)". The new `ModelRejected` end produces `api-error` with no `APIErrorRetries`, and `doc.go:123` says so. Fix: append "or a turn end the engine marks `ModelRejected`" to that clause.

2. **LOW, consistency.** `internal/shuttleengine/wait.go:8` lists the exits of `Wait` as "OutcomeAPIError for a spec with APIErrorRetries". The same new path contradicts it. Fix: add the `ModelRejected` case to that list.

**Tool calls**
1. Grepped for `--effort` across code and docs to find callers of the removed flag.
2. Grepped for `shuttle run ... --model` and `"--model"` outside tests to find callers of the changed flag.
3. Grepped for "committed Codex", "cannot open a session" and the "APIErrorRetries only" wording to find stale prose.
4. Grepped for "assist model key" and "clear the round" outside `_lym` and `.lym`.
5. Grepped `internal/modelspec` and `contracts` for the `haiku` alias, to check that the sandbox suite's `--model haiku` still resolves.
