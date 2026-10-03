# Board

Entries grouped by tier, then by dependency layer.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks

Concrete and claimable; only an entry here can run.

### Layer A

Waits on nothing open; can start now, in parallel.

1. **Verify localizes failures, gates Merriam once, and Publish never runs on a dirty tree** — [`verify-publish-hardening`](design-verify-publish-hardening.md) · feature
   - Cards verify with the integration tests and repo scans they touch; the plan verify runs once as a gate on Merriam, which fixes a failure in a fork and waits on forks instead of polling; Publish never verifies or pushes a dirty tree; verify is never skipped after a crash; every agent prompt goes through a file
   - **Before:** `pattern-init`
1. **Clean pair lifecycle: no leftover sessions, branches, sockets or races** — [`pair-lifecycle`](design-pair-lifecycle.md) · feature · running · Loom-Preflight
   - fabric remove ends the pair's tmux session, finishes half-removed pairs and deletes warp branches on origin; loom start never races a dying driver
   - **Before:** `pattern-init`
1. **Review convergence: the judge decides converged, continue or circling from fresh reviews** — [`review-finding-class`](design-review-finding-class.md) · feature
   - The Bouncer judge stops reviewing the artifacts and judges convergence from facts Go hands it (counts, severities, trend, reopened ledger keys), on its own sonnet[medium] key; at least one fresh round, circling escalates; plus finding classes
   - **Before:** `pattern-init`

### Layer B

Starts when every entry it names under After is done.

1. **PATTERN for loomyard, manifest/ retired, plus repo conventions** — [`pattern-init`](design-pattern-init.md) · feature
   - CONSTRAINTS.md becomes _lyx/PATTERN.md so agents get the constraints directive; manifest/ retires into bolt and package docs; warp/weft scan beyond Go code; model specs in short form; role constants; ide workspace cleanups
   - **After:** `verify-publish-hardening`, `pair-lifecycle`, `review-finding-class`

## Next Up

Planned next, but not yet concretized.

## Notes

Not tasks: ideas and observations, merged into a task when one is promoted.

### Layer A

Waits on nothing open; can start now, in parallel.

1. **Batten teardown deletes an llm-driven child's friction notes** — [`batten-llm-friction`](design-batten-llm-friction.md) · feature
   - [batten][friction] Parked: #274 — only matters once batten runs llm-driven children
1. **Convergence: measure hub and driver interventions, fix at the source, hardening freezes** — [`convergence-plan`](design-convergence-plan.md) · feature
   - [process][testing] Plan for lyx to converge rather than accumulate bugs: per-run intervention log, real-wiring tests, feature freezes
1. **An MCP server over lyx exposing the verbs agents need** — [`lyx-mcp`](design-lyx-mcp.md) · feature
   - [agents][cli] A few coarse MCP tools over lyx, chosen by measured agent use and exposed per role, so agents see typed schemas instead of guessing CLI shapes
1. **creel: a mailbox between named lyx processes** — [`creel`](design-creel.md) · feature
   - [reed][messaging] Mail store and verbs on top of agent-naming's names; deferred while Claude's own SendMessage is enough
1. **One-screen overview of every run in the hub, in its own tmux window** — [`runs-overview`](design-runs-overview.md) · feature
   - [orch][reed][ui] One line per run with state, producer and age, instead of polling each run, shown without squeezing the orch pane (likely reed own-window anchoring)
1. **Discussion advisor: a dedicated agent answers Discussion-Write's questions live** — [`discussion-advisor`](design-discussion-advisor.md) · feature
   - [discussion][agents] A second agent spawned beside Discussion-Write studies the task and code and answers its design questions as they arise, a real two-agent dialogue
1. **Plan cards name the members they change as glyphs, after lyx's use of glyphs is tested systematically** — [`plan-glyph-precision`](design-plan-glyph-precision.md) · feature
   - [plan][glyph][quarry] Plan-Write lists file paths instead of member glyphs because its stencil shows and allows it, and handle-unreferenced forces invented glyphs; test lyx's use of quarry glyphs systematically first
   - **Before:** `card-batching`
1. **Kick-start pack: pre-resolved glyph spans for implementer and Burler** — [`kick-start-pack`](design-kick-start-pack.md) · feature
   - [webster][glyph] Someday (roadmap): #227, #228 — measured win, committed, not yet scheduled
1. **Prompt-cache TTL per agent role** — [`cache-ttl-per-role`](design-cache-ttl-per-role.md) · feature
   - [agents][cost] Parked: workers on 5m cache TTL, waiting sessions on 1h — operator deciding
1. **Orch owns the run loop: detached batten start, a lyx watch, one land command, --continue, a committed role prompt** — [`orch-owns-the-loop`](design-orch-owns-the-loop.md) · feature
   - [orch][batten] Replace the hub scratch scripts and remembered procedure with lyx itself, so an orch session runs the loop from its committed role prompt alone and resumes after a reboot
1. **Parent-review gate: wait bound off the boundary, no mid-turn sends, live smoke** — [`parent-review-hardening`](design-parent-review-hardening.md) · feature
   - [parent-review][orch] Close the cases the first parent-review gate left open: expiry while the writer is mid-turn, sends into a running turn after restart, poll cost, and one live smoke run
1. **lyx orch runs for days without hand repair** — [`orch-live-hardening`](design-orch-live-hardening.md) · feature
   - [orch] Adopt keeps watchers, permission modes mean what they say, attach keeps the orch window size, large-session adopt is tested, status reads the current session
1. **Hardener: an autonomous behavior-review campaign that replaces crucible** — [`hardener`](design-hardener.md) · feature
   - [testing] Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges
1. **Orch and batten run from the operator's own long-lived worktree, not only the prime** — [`orch-any-worktree`](design-orch-any-worktree.md) · feature
   - [orch][batten][fabric] Every orch and batten verb refuses outside the prime; let a long-lived personal worktree be an orch's home, with pairs branching from and landing into its branch
1. **Orch cycle mode as config: clear or compact** — [`orch-cycle-compact`](design-orch-cycle-compact.md) · feature
   - [orch][config] Cycling by /compact keeps the session and its Remote Control link; make clear/compact a config option
1. **Reopen the prime VS Code window with one short command** — [`ide-reopen-prime`](design-ide-reopen-prime.md) · feature
   - [ide] A snap refresh left the prime window wrong; lyx ide spawn needs the slug and fails from the hub root
1. **lyx ide open: launch VS Code without writing anything** — [`ide-open-verb`](design-ide-open-verb.md) · feature
   - [ide][batten] spawn configures and launches at once and overwrites a driven pair's attach chain; add a write-nothing open verb
1. **Pairs as folders in the prime's hub workspace** — [`ide-workspace-pairs`](design-ide-workspace-pairs.md) · design
   - [ide][fabric] Idea, undecided: fabric adds and removes pairs in the prime workspace's folders; _portals stays
1. **lyx board upsert reads the body from a file or stdin** — [`board-body-file`](design-board-body-file.md) · feature
   - [board][cli] Editing one section means splicing JSON in a shell one-liner that breaks on quoting
1. **A labels field on board entries, and tier 2 retired** — [`board-labels`](design-board-labels.md) · feature
   - [board] Labels live as a bracketed brief prefix today; a real field with a filter, and drop the unused tier 2
1. **The orch relies on driver messages, not a scratch watcher** — [`orch-no-watcher`](design-orch-no-watcher.md) · design
   - [orch][batten] The hub watches runs with a hand-written .scratch/hub/watch.sh; drivers already message their parent, so question the watcher, and close any real gap in lyx code

### Layer B

Starts when every entry it names under After is done.

1. **A batchifier that groups small related cards into one Webster batch** — [`card-batching`](design-card-batching.md) · feature
   - [webster][batcher] Only the identity batchifier exists; a mechanical batchifier groups cards from their Create/Edit/Uses fields once cards name members as glyphs
   - **After:** `plan-glyph-precision`
