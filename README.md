# Board

Entries grouped by tier, then by dependency layer.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Planned

Concretized and claimable.

### Layer A

Waits on nothing open; can start now, in parallel.

1. **Verify localizes failures and Publish never runs on a dirty tree** — [`verify-publish-hardening`](design-verify-publish-hardening.md) · feature
   - Cards verify with the integration tests and repo scans they touch, Publish never verifies or pushes a dirty tree, verify is never skipped after a crash, and every agent prompt goes through a file
   - **Before:** `pattern-init`
1. **Clean pair lifecycle: no leftover sessions, branches, sockets or races** — [`pair-lifecycle`](design-pair-lifecycle.md) · feature
   - fabric remove ends the pair's tmux session, finishes half-removed pairs and deletes warp branches on origin; loom start never races a dying driver; tests stop leaking tmux sockets
   - **Before:** `pattern-init`
1. **Retire manifest/: Someday into bolt, durable content into package docs** — [`manifest-retire`](design-manifest-retire.md) · feature
   - Triage roadmap.md's Someday into bolt's tiers, move the durable design content into package docs, fix inbound links and delete manifest/
   - **Before:** `pattern-init`
1. **Review convergence: the judge decides converged, continue or circling from fresh reviews** — [`review-finding-class`](design-review-finding-class.md) · feature
   - The Bouncer judge stops reviewing the artifacts and judges convergence from facts Go hands it (counts, severities, trend, reopened ledger keys), on its own sonnet[medium] key; at least one fresh round, circling escalates; plus finding classes
   - **Before:** `pattern-init`
1. **Webster: verify is a gate on Merriam, Merriam waits instead of polling, and its strand is named webster** — [`webster-verify-gate`](design-webster-verify-gate.md) · feature
   - The plan verify runs once, as a shuttle gate on Merriam that injects failures back for Merriam to fix in a fork; Merriam waits on fork notifications; its strand is named webster

### Layer B

Starts when every entry it names under After is done.

1. **PATTERN for loomyard, plus repo conventions** — [`pattern-init`](design-pattern-init.md) · feature
   - CONSTRAINTS.md becomes _lyx/PATTERN.md so agents get the constraints directive; warp/weft scan beyond Go code; model specs in short form; ide workspace cleanups
   - **After:** `verify-publish-hardening`, `pair-lifecycle`, `manifest-retire`, `review-finding-class`

### Independent

Depends on nothing and nothing depends on it, by design.

1. **Webster recovers from a re-begin, its own canonicalization and an integration regression** — [`webster-recovery`](design-webster-recovery.md) · bug · running · PR-Gate
   - Fix #329, #330, #332: only landed batches count as built, webster's own card rewrites update their hashes, forks fix what their card broke and a bisected regression gets a recovery fork
1. **A run's parent is the worktree it is seeded from; batten survives pauses; parent-review loops** — [`run-parentage`](design-run-parentage.md) · bug · running · Webster-Burler
   - Fix #341, #338: parent from the seeding worktree, batten waits through paused/blocked, parent-review reviews every rewrite up to a cap

## Next Up

Planned next, but not yet concretized.

## Someday

Loose ideas.

### Layer A

Waits on nothing open; can start now, in parallel.

1. **Batten teardown deletes an llm-driven child's friction notes** — [`batten-llm-friction`](design-batten-llm-friction.md) · feature
   - Parked: #274 — only matters once batten runs llm-driven children
1. **Convergence: measure hub and driver interventions, fix at the source, hardening freezes** — [`convergence-plan`](design-convergence-plan.md) · feature
   - Plan for lyx to converge rather than accumulate bugs: per-run intervention log, real-wiring tests, feature freezes
1. **Test suite: cut change amplification** — [`test-suite-change-amplification`](design-test-suite-change-amplification.md) · feature
   - Test code is ~3x production and growing faster; a small prod change forces edits in many test files, each one context an implementing agent must read. Survey and plan in GitHub #323, work split into #324-#327.
1. **An MCP server over lyx exposing the verbs agents need** — [`lyx-mcp`](design-lyx-mcp.md) · feature
   - A few coarse MCP tools over lyx, chosen by measured agent use and exposed per role, so agents see typed schemas instead of guessing CLI shapes
1. **creel: a mailbox between named lyx processes** — [`creel`](design-creel.md) · feature
   - Mail store and verbs on top of agent-naming's names; deferred while Claude's own SendMessage is enough
1. **One-screen overview of every run in the hub** — [`runs-overview`](design-runs-overview.md) · feature
   - One line per run with state, producer and age, instead of polling each run
1. **Discussion advisor: a dedicated agent answers Discussion-Write's questions live** — [`discussion-advisor`](design-discussion-advisor.md) · feature
   - A second agent spawned beside Discussion-Write studies the task and code and answers its design questions as they arise, a real two-agent dialogue
1. **Plan cards name the members they change as glyphs, after the glyph library is tested properly** — [`plan-glyph-precision`](design-plan-glyph-precision.md) · feature
   - Plan-Write lists file paths instead of member glyphs because its stencil shows and allows it, and handle-unreferenced forces invented glyphs; test the glyph library at scale first
1. **Kick-start pack: pre-resolved glyph spans for implementer and Burler** — [`kick-start-pack`](design-kick-start-pack.md) · feature
   - Someday (roadmap): #227, #228 — measured win, committed, not yet scheduled
1. **Prompt-cache TTL per agent role** — [`cache-ttl-per-role`](design-cache-ttl-per-role.md) · feature
   - Parked: workers on 5m cache TTL, waiting sessions on 1h — operator deciding
