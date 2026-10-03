# Board

Entries grouped by tier, then by dependency layer.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Planned

Concretized and claimable.

### Layer A

Waits on nothing open; can start now, in parallel.

1. **Test suite, part 2: derived guards, decoupled recipe tests, no leaked tmux sockets** — [`test-suite-derived-guards`](design-test-suite-derived-guards.md) · feature · running · Plan-Burler
   - One invariant-scan harness; registry and CLI-tree tests derived from the real registries; recipe/plan/stencil tests decoupled from their sources; fixtures clean up tmux sockets
   - **Before:** `verify-publish-hardening`, `pair-lifecycle`, `review-finding-class`

### Layer B

Starts when every entry it names under After is done.

1. **Verify localizes failures, gates Merriam once, and Publish never runs on a dirty tree** — [`verify-publish-hardening`](design-verify-publish-hardening.md) · feature
   - Cards verify with the integration tests and repo scans they touch; the plan verify runs once as a gate on Merriam, which fixes a failure in a fork and waits on forks instead of polling; Publish never verifies or pushes a dirty tree; verify is never skipped after a crash; every agent prompt goes through a file
   - **After:** `test-suite-derived-guards`
   - **Before:** `pattern-init`
1. **Clean pair lifecycle: no leftover sessions, branches, sockets or races** — [`pair-lifecycle`](design-pair-lifecycle.md) · feature
   - fabric remove ends the pair's tmux session, finishes half-removed pairs and deletes warp branches on origin; loom start never races a dying driver
   - **After:** `test-suite-derived-guards`
   - **Before:** `pattern-init`
1. **Review convergence: the judge decides converged, continue or circling from fresh reviews** — [`review-finding-class`](design-review-finding-class.md) · feature
   - The Bouncer judge stops reviewing the artifacts and judges convergence from facts Go hands it (counts, severities, trend, reopened ledger keys), on its own sonnet[medium] key; at least one fresh round, circling escalates; plus finding classes
   - **After:** `test-suite-derived-guards`
   - **Before:** `pattern-init`

### Layer C

Starts when every entry it names under After is done.

1. **PATTERN for loomyard, manifest/ retired, plus repo conventions** — [`pattern-init`](design-pattern-init.md) · feature
   - CONSTRAINTS.md becomes _lyx/PATTERN.md so agents get the constraints directive; manifest/ retires into bolt and package docs; warp/weft scan beyond Go code; model specs in short form; role constants; ide workspace cleanups
   - **After:** `verify-publish-hardening`, `pair-lifecycle`, `review-finding-class`

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
1. **An MCP server over lyx exposing the verbs agents need** — [`lyx-mcp`](design-lyx-mcp.md) · feature
   - A few coarse MCP tools over lyx, chosen by measured agent use and exposed per role, so agents see typed schemas instead of guessing CLI shapes
1. **creel: a mailbox between named lyx processes** — [`creel`](design-creel.md) · feature
   - Mail store and verbs on top of agent-naming's names; deferred while Claude's own SendMessage is enough
1. **One-screen overview of every run in the hub, in its own tmux window** — [`runs-overview`](design-runs-overview.md) · feature
   - One line per run with state, producer and age, instead of polling each run, shown without squeezing the orch pane (likely reed own-window anchoring)
1. **Discussion advisor: a dedicated agent answers Discussion-Write's questions live** — [`discussion-advisor`](design-discussion-advisor.md) · feature
   - A second agent spawned beside Discussion-Write studies the task and code and answers its design questions as they arise, a real two-agent dialogue
1. **Plan cards name the members they change as glyphs, after lyx's use of glyphs is tested systematically** — [`plan-glyph-precision`](design-plan-glyph-precision.md) · feature
   - Plan-Write lists file paths instead of member glyphs because its stencil shows and allows it, and handle-unreferenced forces invented glyphs; test lyx's use of quarry glyphs systematically first
   - **Before:** `card-batching`
1. **Kick-start pack: pre-resolved glyph spans for implementer and Burler** — [`kick-start-pack`](design-kick-start-pack.md) · feature
   - Someday (roadmap): #227, #228 — measured win, committed, not yet scheduled
1. **Prompt-cache TTL per agent role** — [`cache-ttl-per-role`](design-cache-ttl-per-role.md) · feature
   - Parked: workers on 5m cache TTL, waiting sessions on 1h — operator deciding
1. **Orch owns the run loop: detached batten start, a lyx watch, one land command, --continue, a committed role prompt** — [`orch-owns-the-loop`](design-orch-owns-the-loop.md) · feature
   - Replace the hub scratch scripts and remembered procedure with lyx itself, so an orch session runs the loop from its committed role prompt alone and resumes after a reboot
1. **Parent-review gate: wait bound off the boundary, no mid-turn sends, live smoke** — [`parent-review-hardening`](design-parent-review-hardening.md) · feature
   - Close the cases the first parent-review gate left open: expiry while the writer is mid-turn, sends into a running turn after restart, poll cost, and one live smoke run
1. **lyx orch runs for days without hand repair** — [`orch-live-hardening`](design-orch-live-hardening.md) · feature
   - Adopt keeps watchers, permission modes mean what they say, attach keeps the orch window size, large-session adopt is tested, status reads the current session
1. **Hardener: an autonomous behavior-review campaign that replaces crucible** — [`hardener`](design-hardener.md) · feature
   - Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges

### Layer B

Starts when every entry it names under After is done.

1. **A batchifier that groups small related cards into one Webster batch** — [`card-batching`](design-card-batching.md) · feature
   - Only the identity batchifier exists; a mechanical batchifier groups cards from their Create/Edit/Uses fields once cards name members as glyphs
   - **After:** `plan-glyph-precision`
