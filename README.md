# Board

Entries grouped by tier, in dependency order within each tier.

## Planned

Concretized and claimable.

1. **Verify localizes failures and Publish never runs on a dirty tree** — `verify-publish-hardening` · feature
   Cards verify with the integration tests and repo scans they touch, Publish never verifies or pushes a dirty tree, verify is never skipped after a crash, and every agent prompt goes through a file [design](design-verify-publish-hardening.md)
1. **Clean pair lifecycle: no leftover sessions, branches, sockets or races** — `pair-lifecycle` · feature
   fabric remove ends the pair's tmux session, finishes half-removed pairs and deletes warp branches on origin; loom start never races a dying driver; tests stop leaking tmux sockets [design](design-pair-lifecycle.md)
1. **Parent review as a step in Discussion-Write's gate** — `parent-review` · feature
   Port of millhouse orch-review/orch-wait into Discussion-Write's producer gate: after the mechanical validator, a reviewer (the parent for now) reviews once through a lyx verb pair and the live writer addresses it; a new pass_on_cap gate flag lets the run through afterwards [design](design-parent-review.md)
1. **Retire manifest/: Someday into bolt, durable content into package docs** — `manifest-retire` · feature
   Triage roadmap.md's Someday into bolt's tiers, move the durable design content into package docs, fix inbound links and delete manifest/ [design](design-manifest-retire.md)
1. **Review convergence: the judge decides converged, continue or circling from fresh reviews** — `review-finding-class` · feature
   The Bouncer judge stops reviewing the artifacts and judges convergence from facts Go hands it (counts, severities, trend, reopened ledger keys), on its own sonnet[medium] key; at least one fresh round, circling escalates; plus finding classes [design](design-review-finding-class.md)
1. **lyx orch: ready to host the real hub orchestrator** — `orch-live-ready` · feature
   orch is deployed but untested live and lacks what the hub needs: bypass permissions, the Agent tool, adoption of a running session, a soft threshold [design](design-orch-live-ready.md)
1. **PATTERN for loomyard, plus repo conventions** — `pattern-init` · feature
   CONSTRAINTS.md becomes _lyx/PATTERN.md so agents get the constraints directive; warp/weft scan beyond Go code; model specs in short form; ide workspace cleanups [design](design-pattern-init.md)
   After `parent-review`, `verify-publish-hardening`, `pair-lifecycle`, `manifest-retire`, `review-finding-class`.

## Next Up

Planned next, but not yet concretized.

## Someday

Loose ideas.

1. **Batten teardown deletes an llm-driven child's friction notes** — `batten-llm-friction` · feature
   Parked: #274 — only matters once batten runs llm-driven children [design](design-batten-llm-friction.md)
1. **Convergence: measure hub and driver interventions, fix at the source, hardening freezes** — `convergence-plan` · feature
   Plan for lyx to converge rather than accumulate bugs: per-run intervention log, real-wiring tests, feature freezes [design](design-convergence-plan.md)
1. **Test suite: cut change amplification** — `test-suite-change-amplification` · feature
   Test code is ~3x production and growing faster; a small prod change forces edits in many test files, each one context an implementing agent must read. Survey and plan in GitHub #323, work split into #324-#327. [design](design-test-suite-change-amplification.md)
1. **An MCP server over lyx exposing the verbs agents need** — `lyx-mcp` · feature
   A few coarse MCP tools over lyx, chosen by measured agent use and exposed per role, so agents see typed schemas instead of guessing CLI shapes [design](design-lyx-mcp.md)
1. **creel: a mailbox between named lyx processes** — `creel` · feature
   Mail store and verbs on top of agent-naming's names; deferred while Claude's own SendMessage is enough [design](design-creel.md)
1. **One-screen overview of every run in the hub** — `runs-overview` · feature
   One line per run with state, producer and age, instead of polling each run [design](design-runs-overview.md)
1. **Discussion advisor: a dedicated agent answers Discussion-Write's questions live** — `discussion-advisor` · feature
   A second agent spawned beside Discussion-Write studies the task and code and answers its design questions as they arise, a real two-agent dialogue; after parent-review [design](design-discussion-advisor.md)
1. **Plan cards name the members they change as glyphs, after the glyph library is tested properly** — `plan-glyph-precision` · feature
   Plan-Write lists file paths instead of member glyphs because its stencil shows and allows it, and handle-unreferenced forces invented glyphs; test the glyph library at scale first [design](design-plan-glyph-precision.md)
1. **Kick-start pack: pre-resolved glyph spans for implementer and Burler** — `kick-start-pack` · feature
   Someday (roadmap): #227, #228 — measured win, committed, not yet scheduled [design](design-kick-start-pack.md)
1. **Prompt-cache TTL per agent role** — `cache-ttl-per-role` · feature
   Parked: workers on 5m cache TTL, waiting sessions on 1h — operator deciding [design](design-cache-ttl-per-role.md)
