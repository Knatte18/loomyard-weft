# Card 1 — engine-prompt-cache-lifetime

**Edit:**
- `internal/shuttleengine#Engine`
- `internal/shuttleengine/claudeengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/shuttleengine/claudeengine#Claude.PromptCacheLifetime` -> `func (c *Claude) PromptCacheLifetime(spec shuttleengine.Spec, cfg shuttleengine.Config) (time.Duration, error)`
- `plan:internal/shuttleengine#fakeEngine.PromptCacheLifetime` -> `func (e *fakeEngine) PromptCacheLifetime(spec Spec, cfg Config) (time.Duration, error)`
- `plan:internal/testkit/shuttlefake#Engine.PromptCacheLifetime` -> `func (e *Engine) PromptCacheLifetime(spec shuttleengine.Spec, cfg shuttleengine.Config) (time.Duration, error)`
- `plan:internal/shuttleengine/claudeengine#TestPromptCacheLifetime` -> `func TestPromptCacheLifetime(t *testing.T)`

**Uses:**
- `internal/shuttleengine/claudeengine#resolvePromptCacheTTL`

**Intent:**
D1's provider-neutral seam: `shuttleengine.Engine` gains `PromptCacheLifetime(spec Spec, cfg Config) (time.Duration, error)`, how long the provider keeps a session's prompt cache warm while the session sits idle, for a session started from `spec` under `cfg`.
Its doc says zero means the provider knows no warm lifetime, so a caller keeps nothing warm, and an error means `cfg` cannot be resolved for `spec`.
No cache lifetime is spelled outside the provider (`PATTERN-shuttle-provider-seam`).

`Claude.PromptCacheLifetime` resolves the role's TTL with `resolvePromptCacheTTL(spec.Role, cfg)`, the value `Prepare` puts on the pane line, and returns it parsed as a duration; a resolution error is returned unchanged, with its shuttle.yaml advice.
`claudeengine/doc.go`'s prompt-cache paragraph names the method.

The two fakes, `fakeEngine` in the shuttleengine tests and `shuttlefake.Engine`, answer zero and no error, so no test-built gate pings.

Tests:
`TestPromptCacheLifetime`, in `prepare_test.go` beside the TTL rows and the `ttlConfig` helper, is a table over `ttlConfig` covering what the method adds over the shared resolver, the parse to a duration and the passed-through error: the 5m default answers five minutes; a role mapped to `1h` answers an hour; an invalid default answers an error naming `claude_prompt_cache_ttl`.
`test-budget.yaml`'s `internal/shuttleengine/claudeengine` row rises by one for it.

**ImpactSummary:** one method on the Engine interface, implemented by Claude and the two test fakes; no existing method or caller changes.
