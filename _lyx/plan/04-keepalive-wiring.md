# Card 4 — keepalive-wiring

**Create:**
- `plan:internal/loomengine#DiscussionWriterRole` -> `func DiscussionWriterRole(cfg Config) string`
- `plan:internal/loomcli#keepAliveInterval` -> `func keepAliveInterval(lifetime, waitBound time.Duration) time.Duration`
- `plan:internal/loomcli#TestNewParentReviewConfig_KeepAliveInterval` -> `func TestNewParentReviewConfig_KeepAliveInterval(t *testing.T)`

**Edit:**
- `internal/loomcli#newParentReviewConfig` -> `func newParentReviewConfig(location *lyxcwd.Location, reedCfg reedengine.Config, cfg loomengine.Config, stencilsDir string, engine shuttleengine.Engine, shuttleCfg shuttleengine.Config) (parentreview.GateConfig, error)`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#TestNewParentReviewConfig_ReviewerFromResolver`
- `internal/loomcli#TestSmokeParentReview_NoticeReachesTheWriterPaneAndApproveLetsTheRunThrough`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/loomengine#ParentReviewKeepAlivePrompt`

**Intent:**
D1's wiring: the loom wiring sets the keep-alive interval from the writer's prompt-cache lifetime, which only the engine answers.

`DiscussionWriterRole` returns the role of the session the parent-review gate prompts: the seats chair's `seatengine.SeatRole(discussionRole, seatengine.RoleChair)` when `discussion_producer` is `seats`, and `discussionRole` for `single` and `twill`.
Advisors are out, since they take no turn after the chair's record exists.

`keepAliveInterval` returns `lifetime` less one minute, and zero when `lifetime` reaches `waitBound` or is one minute or less.
At the 5m default that is 4 minutes, and a writer whose role is mapped to `1h` gets no ping at the default 60-minute bound.

`newParentReviewConfig` takes the engine and the shuttle config, asks `engine.PromptCacheLifetime` for a `shuttleengine.Spec` carrying `DiscussionWriterRole(cfg)` as its role, and sets `KeepAlive` to `keepAliveInterval` of the answer and `WaitBound`.
A lifetime error logs a Warn naming the error and leaves `KeepAlive` zero, since the writer's own spawn refuses the same config with its advice.
It sets `RenderKeepAlive` to `loomengine.ParentReviewKeepAlivePrompt` over the stencils directory.
`loomCLI.wire` passes `claudeEngine` and `shuttleCfg`, both already built before the call.
The two existing callers in tests pass a `shuttlefake` engine or the smoke test's own engine with a loaded shuttle config, which answer zero and leave pings off.

Tests:
`TestNewParentReviewConfig_KeepAliveInterval` builds the config over the real `claudeengine.NewFromConfig` engine on the location `TestNewParentReviewConfig_ReviewerFromResolver` uses, as a table over `discussion_producer`, the shuttle config's TTL default and role map, and the wait bound:
`single` at the 5m default with a 60-minute bound gives 4 minutes; `single` with `discussion` mapped to `1h` gives zero; `seats` with the chair's role mapped to `1h` gives zero, and at the default gives 4 minutes; an invalid TTL gives zero.
Each row with a positive interval asserts `RenderKeepAlive` is set.
`test-budget.yaml`'s `internal/loomcli` row rises by one for the new test.

**ImpactSummary:** newParentReviewConfig gains two arguments, threaded from wire and its two test callers; pings switch on in production at the writer's cache lifetime less a minute.
