# Card 2 — keepalive-stencil

**Create:**
- `contracts/stencils/loom/loom-template-parent-review-keepalive.md`
- `plan:contracts/stencils#LoomTemplateParentReviewKeepalive` -> `var LoomTemplateParentReviewKeepalive []byte`
- `plan:internal/loomengine#parentReviewKeepAliveStencil` -> `const parentReviewKeepAliveStencil = "loom-template-parent-review-keepalive"`
- `plan:internal/loomengine#ParentReviewKeepAlivePrompt` -> `func ParentReviewKeepAlivePrompt(stencilsDir string, ping int) (string, error)`
- `plan:internal/loomengine#TestParentReviewKeepAlivePrompt_OneLineNamingPing` -> `func TestParentReviewKeepAlivePrompt_OneLineNamingPing(t *testing.T)`

**Edit:**
- `contracts/stencils#entries`
- `contracts/stencils#wordingClaims`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `cmd/lyx/testdata/test-budget.yaml`

**Prosa:**
- `contracts/stencils/loom/loom-template-discussion.md`

**Intent:**
D1's keep-alive line is a one-line stencil read at call time (`PATTERN-stencil-ownership`).

`loom-template-parent-review-keepalive.md` opens with a comment in the delivery stencil's style naming its renderer, its one marker `{{.ping}}` and the one-line rule.
Its line tells the writer this is keep-alive number `{{.ping}}` while the parent's review is pending, and to change nothing, run nothing and end its turn.
The ping number makes each ping's text distinct, so shuttle's pending-send record never mistakes a ping for an earlier one already sent.
`stencils.go` embeds it as `LoomTemplateParentReviewKeepalive`, and `entries` registers it after `loom-template-parent-review-brief`.

`ParentReviewKeepAlivePrompt` reads the stencil through `stencilstore`, fills `ping` with the number, trims it and refuses a render holding a line break, as `ParentReviewDeliveryPrompt` does; `parentreview.go`'s file header names the third prompt.

`loom-template-discussion.md`'s `## Messages the gate may send after you end your turn` section, which the seats chair also receives through its include, says the gate may send one of three messages and none changes the design.
A third bullet names the keep-alive: it changes nothing and asks only that the writer end its turn.
It points at the message without restating its text (`PATTERN-producer-pointer-rule`).

Tests:
`TestParentReviewKeepAlivePrompt_OneLineNamingPing` renders pings 1 and 2 from the seeded stencils directory and asserts each is one line, names its number, and the two differ; a stencil holding a line break refuses.
`TestStencils_TopLevelMarkers` gains a row pinning the keep-alive stencil's marker set to `ping`.
`wordingClaims` gains a row for the keep-alive stencil claiming "end your turn" and that it changes nothing, and its `loom-template-discussion.md` row gains a claim on the keep-alive bullet in the messages section.
`test-budget.yaml`'s `internal/loomengine` row rises by one for the new test.

**ImpactSummary:** one new registered stencil, its renderer and claims, and one added bullet in the discussion stencil; no existing prompt's text changes beyond that section.
