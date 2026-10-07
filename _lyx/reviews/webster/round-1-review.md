---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/shuttleengine/claudeengine/command.go:147
    summary: resolvePromptCacheTTL looks up an empty role in the map, so a map key of empty string overrides the default although D3 and the function's own doc say an empty role takes the default
  - id: F2
    severity: LOW
    class: consistency
    location: internal/shuttleengine/claudeengine/prepare_test.go:83
    summary: TestPrepare_RejectedBeforeArtifacts treats a zero Config as "use the template" unless wantTTLAdvice is set, a hidden sentinel, where card 3 and the sibling table give each row its cfg explicitly
  - id: F3
    severity: LOW
    class: consistency
    location: internal/shuttleengine/claudeengine/prepare_test.go:72
    summary: the ttl_default_wrong_case and ttl_default_bare_number rows assert only the offending value, not the key, though card 3 says each TTL row's wantErrContains names the key
  - id: F4
    severity: NIT
    class: consistency
    location: internal/shuttleengine/claudeengine/command.go:128
    summary: the resolvePromptCacheTTL doc comment chains two "so" clauses in one sentence, which reads as one muddled sentence
---

Review range: `status.json` gives `product.parent: main`, and there is no `_lyx/loom/rework/` directory, so the range is `git diff $(git merge-base main HEAD)..HEAD`: the three card commits.
`go build ./...`, `go vet` under `integration`, `tmux` and `llm`, `go test ./...` and the plan's integration run all pass, as does `lyx loom lint-comments --base <merge-base>`.
The `test-budget.yaml` rows match the function counts (141 for `internal/shuttleengine`, one new test per package).
No card carries a Rename or Delete group, so the per-type mechanical checks have nothing to run beyond the build and tests.

### [LOW] An empty role can still match a map entry

**Location:** `internal/shuttleengine/claudeengine/command.go:147`
**Issue:** The shared decision D3 and the function's doc comment say an empty role takes `claude_prompt_cache_ttl`.
The code does `cfg.ClaudePromptCacheTTLRoles[role]` unconditionally, so a `shuttle.yaml` whose map holds an entry with an empty key (a valid, validated entry) gives every empty-role spawn that entry's value instead of the default.
**Fix:** Look the map up only when `role != ""`, and add a row to `TestPrepare_ThreadsSpecIntoLaunchCmds` (an empty role with a map holding an empty key still gets the default).

### [LOW] A zero Config is silently swapped for the template in the rejected-before-artifacts test

**Location:** `internal/shuttleengine/claudeengine/prepare_test.go:83`
**Issue:** `reflect.DeepEqual(cfg, shuttleengine.Config{}) && !tt.wantTTLAdvice` makes the meaning of the `cfg` column depend on an unrelated column.
Card 3 gives the existing rows the template config in that column, and `TestPrepare_ThreadsSpecIntoLaunchCmds` does so explicitly on every row.
**Fix:** Set `cfg: templateConfig(t)` on each non-TTL row and drop the `reflect` fallback and its import, so the `ttl_default_empty` row's `shuttleengine.Config{}` means what it says.

### [LOW] Two TTL rows do not name the key in their expected error

**Location:** `internal/shuttleengine/claudeengine/prepare_test.go:72`
**Issue:** `ttl_default_wrong_case` and `ttl_default_bare_number` assert `"1H"` and `"5"`, the value only.
Card 3 requires each TTL row's `wantErrContains` to name the key; the message names it, but nothing pins it for these rows.
**Fix:** Assert `claude_prompt_cache_ttl "1H"` and `claude_prompt_cache_ttl "5"`, the key and value as the error renders them.

### [NIT] Muddled sentence in the resolvePromptCacheTTL doc comment

**Location:** `internal/shuttleengine/claudeengine/command.go:128`
**Issue:** "...in sorted key order so the reported entry is deterministic, so one bad entry refuses every role..." stacks two "so" clauses, and the second reads as a consequence of the sort order.
**Fix:** Split it into two sentences, one per line: the validation and its sorted order, then that one bad entry refuses every role until `shuttle.yaml` is fixed.

## Settled focus questions

- `wait_test.go` and `envkit.go` changes are required by card 1's behavior, not scope creep: `Config` gained a map field, so `cfg == (Config{})` no longer compiles, and `envkit.NilSeams` flags every nil map, which the Config inside `WebsterDeps.ShuttleCfg` legitimately is.
  The `envkit` diff beyond the one new `nilLegal` row is gofmt realignment.
- `LoadConfig` carries the map whole through `ConfigOpenMaps()...` and adds no TTL validation; `TestLoadConfig_PromptCacheTTLKeys` pins an `orch`-only map surviving with `driver` not restored.
- `resolvePromptCacheTTL` validates the default, then every map value in sorted key order, rejecting empty, `1H` and `5`; the error names the key, the value, `5m`, `1h` and `shuttle.yaml`; the role match is exact.
- On both lines the TTL wrap sits after the fork-subagent wrap and before the `CLAUDE_ENV_FILE` wrap, and `Prepare` resolves once, after the other validations and before any artifact, and hands the one value to both builders.
- The env var name and the `5m|1h` vocabulary appear only under `claudeengine` in Go code; `shuttleengine` and `configreg` interpret nothing.
- Every `Config` handed to `Prepare` takes its TTL fields from the template helper (`templateConfig`/`ttlConfig`, and `LoadConfig` in `webstercli/smoke_test.go`); the remaining literal `Config{}` values go to `buildSettings` or the fake engine, not `Prepare`.
- No new test duplicates an existing one: `TestLoadConfig_PromptCacheTTLKeys` seeds a file, where `TestLoadConfig_TemplateDefaultsResolve` loads with none, and the log test is the only one reading the log line.
- Docs: `claudeengine/doc.go`, `shuttleengine/doc.go` and `docs/overview.md` state what the code does, including that `FORCE_PROMPT_CACHING_5M` wins and the subagent bucket is untouched.
