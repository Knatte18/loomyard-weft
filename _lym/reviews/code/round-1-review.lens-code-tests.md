```yaml
lens: code-tests
outcome: reported
```

**Per behaviour: the test that pins it**

1. **Template default is `model: opus` and `effort: medium`.**
   - Pinned by `TestLoadConfig_TemplateResolvesWithNoFile` at `internal/orchengine/config_test.go:39-40`.
   - It breaks if either value in `template.yaml` changes.
   - That test calls `LoadConfig`, so it also pins that the default reaches `Config`.
2. **`orchSpec` carries `c.cfg.Model` and `c.cfg.Effort` into the spec's `ModelChoice`.**
   - This line did not change in the diff.
   - Pinned by `internal/orchcli/start_test.go:167`, which sets `opus`/`high`, and `start_test.go:494-495`, which asserts the spec carries `opus`/`high`.
   - A reverted or hard-coded value would fail the asserted `high`.
3. **Does effort `medium` resolve in shuttle's vocabulary (`validateEffort` against `agentconfig.Efforts()`)?**
   - I found no test that feeds `medium` through `validateEffort` or the template.
   - I did not read `effortVocabulary`, so I cannot say whether `medium` is in it.
   - The existing `validateEffort` test, if any, is outside this diff. I did not read `claudeengine`'s tests, so I report no finding on it.
4. **An explicit empty override still means the provider default, as the new template comment claims.**
   - No row in the diff or in the tests I saw loads an explicit `model: ""` over the new default and asserts that it stays empty.
   - `config_test.go:131` shows a worktree file stacking over the default for `effort`, but only with a non-empty value.
   - I did not check how the layer merge treats empty values.
   - That is the comment's claim. The diff adds no new code for it, and it is a pre-existing merge behaviour with no test of its own.

**Findings**

- **LOW, `design`:** the template comment now documents "an explicit empty value uses the provider default", and no test pins that behaviour.
  - Location: `internal/orchengine/template.yaml:1-2`, and the layering tests in `internal/orchengine/config_test.go` (around lines 81 and 131).
  - Evidence: those tests only stack non-empty values.
  - Suggested fix: add one row to the existing layered-load test, in which a layer sets `model: ""` and the loaded `cfg.Model` is empty.
  - The handler or the affected-set lens should confirm this against the merge code first. If the merge does not honor an empty override, this becomes a correctness finding.
- Nothing else is uncovered. The changed assertion in the one existing test is enough for the default. The start-path wiring already has `start_test.go:494-495`. A new start-path test would duplicate it.

**Tool calls (1)**
- `grep` over the orchcli and orchengine tests for `Model`/`ModelChoice` assertions: my context did not show the test files.
