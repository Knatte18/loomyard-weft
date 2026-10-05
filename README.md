# Board

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks

Concrete and claimable; only a task can run.

### Layer A

Waits on nothing open; can start now, in parallel.

1. **Webster audit and verify-gate halts are recoverable by the agent** — [`webster-audit-recovery`](design-webster-audit-recovery.md) · bug, enhancement, webster, shuttle · running · Webster
   - Hung-shell gate wait, unacceptable contract-file findings, garbled way-forward text, recoveries that need denied commands
1. **Orch context cycling and long-run hardening** — [`orch-cycle-hardening`](design-orch-cycle-hardening.md) · bug, enhancement, orch, config, batten · running · Webster
   - Fresh reading after /compact, clear-or-compact as config, and the adopt/attach/permission hardening list
   - **Before:** `lyx-skills-and-stencils`
1. **Bouncer CIRCLING escalates to the parent; loomcli smoke tier green** — [`loom-bouncer-rounds`](design-loom-bouncer-rounds.md) · bug, enhancement, loom, orch · running · Webster
   - CIRCLING judged later on unchanged recurrence and escalated to the parent orchestrator; fix the five failing loomcli smoke tests
   - **Before:** `lyx-skills-and-stencils`
1. **Label descriptions, board CLI gaps, CLI tree exceptions** — [`board-labels-and-skills`](design-board-labels-and-skills.md) · bug, enhancement, board, config, selfreport · running · Publish
   - labels become label→description with lyx board labels; board get --body, merge --body-file, merged issues kept, slug limit in help; fix the five cliTreeFindings exceptions

### Layer B

Starts when every entry it names under After is done.

1. **ly-drive as a stencil, a ly:runs skill, and lyx-injected skill loading** — [`lyx-skills-and-stencils`](design-lyx-skills-and-stencils.md) · enhancement, stencil, orch, shed
   - Driver procedure moves into a stencil, a ly:runs workflow skill, and the orch watcher injects skill loading at start, adopt and after every cycle
   - **After:** `orch-cycle-hardening`, `loom-bouncer-rounds`

## Notes

Not tasks: ideas and observations, merged into a task when one is promoted.

### Bugs

1. **Webster blocks when a Delete target is already gone** — [`webster-delete-already-gone`](design-webster-delete-already-gone.md) · bug, webster
   - begin-batch refuses plan_drifted when an earlier batch already deleted a later card's Delete target
1. **Webster schedules a Delete before its callers are removed** — [`webster-delete-ordering`](design-webster-delete-ordering.md) · bug, webster
   - deriveEdges has no edge from a deleted symbol to the cards editing its callers
1. **Webster begin-batch spends a minute validating the plan per batch** — [`webster-batch-validation-speed`](design-webster-batch-validation-speed.md) · bug, webster, quarry
   - quarry#34 re-parses per target; fix it or validate incrementally; drop the per-batch model assertion
1. **Config changes a task makes never reach the parent** — [`config-reaches-parent`](design-config-reaches-parent.md) · bug, config, fabric, loom
   - Live config is in weft and task weft dies at landing; carry config changes or make the hub step explicit
1. **Webster Master hangs background shells with a flattened heredoc** — [`master-heredoc-hang`](design-master-heredoc-hang.md) · bug, webster, shuttle
   - cat > /tmp/x; python3 - <<EOF … EOF on one line reads stdin forever; seen twice on 2026-10-05 and in #361

### Enhancements

1. **Convergence: measure hub and driver interventions, fix at the source, hardening freezes** — [`convergence-plan`](design-convergence-plan.md) · enhancement
   - Plan for lyx to converge rather than accumulate bugs: per-run intervention log, real-wiring tests, feature freezes
1. **An MCP server over lyx exposing the verbs agents need** — [`lyx-mcp`](design-lyx-mcp.md) · enhancement
   - A few coarse MCP tools over lyx, chosen by measured agent use and exposed per role, so agents see typed schemas instead of guessing CLI shapes
1. **creel: a mailbox between named lyx processes** — [`creel`](design-creel.md) · enhancement, reed
   - Mail store and verbs on top of agent-naming's names; deferred while Claude's own SendMessage is enough
1. **One-screen overview of every run in the hub, in its own tmux window** — [`runs-overview`](design-runs-overview.md) · enhancement, reed, orch
   - One line per run with state, producer and age, instead of polling each run, shown without squeezing the orch pane (likely reed own-window anchoring)
1. **Discussion advisor: a dedicated agent answers Discussion-Write's questions live** — [`discussion-advisor`](design-discussion-advisor.md) · enhancement, shuttle, loom
   - A second agent spawned beside Discussion-Write studies the task and code and answers its design questions as they arise, a real two-agent dialogue
1. **Plan cards name the members they change as glyphs, after lyx's use of glyphs is tested systematically** — [`plan-glyph-precision`](design-plan-glyph-precision.md) · enhancement, loom, quarry
   - Plan-Write lists file paths instead of member glyphs because its stencil shows and allows it, and handle-unreferenced forces invented glyphs; test lyx's use of quarry glyphs systematically first
1. **Kick-start pack: pre-resolved glyph spans for implementer and Burler** — [`kick-start-pack`](design-kick-start-pack.md) · enhancement, webster, quarry
   - Someday (roadmap): #227, #228 — measured win, committed, not yet scheduled
1. **Prompt-cache TTL per agent role** — [`cache-ttl-per-role`](design-cache-ttl-per-role.md) · enhancement, shuttle
   - Parked: workers on 5m cache TTL, waiting sessions on 1h — operator deciding
1. **Orch owns the run loop: detached batten start, a lyx watch, one land command, --continue, a committed role prompt** — [`orch-owns-the-loop`](design-orch-owns-the-loop.md) · enhancement, batten, orch
   - Replace the hub scratch scripts and remembered procedure with lyx itself, so an orch session runs the loop from its committed role prompt alone and resumes after a reboot
1. **Parent-review gate: wait bound off the boundary, no mid-turn sends, live smoke** — [`parent-review-hardening`](design-parent-review-hardening.md) · enhancement, orch
   - Close the cases the first parent-review gate left open: expiry while the writer is mid-turn, sends into a running turn after restart, poll cost, and one live smoke run
1. **Hardener: an autonomous behavior-review campaign that replaces crucible** — [`hardener`](design-hardener.md) · enhancement
   - Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges
1. **A batchifier that groups small related cards into one Webster batch** — [`card-batching`](design-card-batching.md) · enhancement, webster
   - Only the identity batchifier exists; a mechanical batchifier groups cards from their Create/Edit/Uses fields once cards name members as glyphs
1. **Orch and batten run from the operator's own long-lived worktree, not only the prime** — [`orch-any-worktree`](design-orch-any-worktree.md) · enhancement, fabric, batten, orch
   - Every orch and batten verb refuses outside the prime; let a long-lived personal worktree be an orch's home, with pairs branching from and landing into its branch
1. **Reopen the prime VS Code window with one short command** — [`ide-reopen-prime`](design-ide-reopen-prime.md) · enhancement, ide
   - A snap refresh left the prime window wrong; lyx ide spawn needs the slug and fails from the hub root
1. **lyx ide open: launch VS Code without writing anything** — [`ide-open-verb`](design-ide-open-verb.md) · enhancement, ide, batten
   - spawn configures and launches at once and overwrites a driven pair's attach chain; add a write-nothing open verb
1. **Pairs as folders in the prime's hub workspace** — [`ide-workspace-pairs`](design-ide-workspace-pairs.md) · enhancement, ide, fabric
   - Idea, undecided: fabric adds and removes pairs in the prime workspace's folders; _portals stays
1. **Each batten the orch starts runs in its own tmux window** — [`batten-windows`](design-batten-windows.md) · enhancement, reed, batten, orch
   - Batten runs detached and invisible; a window per run makes the window list show what the orch has in flight
1. **Fabric's Windows path behaviour verified on a Windows host** — [`fabric-windows`](design-fabric-windows.md) · enhancement, fabric
   - Unverified after six hardening rounds; needs a Windows host, not design; take it only if Windows becomes relevant
1. **semantic-index: find code by concept, not by literal keyword** — [`semantic-index`](design-semantic-index.md) · enhancement
   - Semantic search over docstrings and comments; doubtful today, since agents now navigate well with grep
1. **VS Code only on request; other ways to watch a run** — [`ide-opt-in`](design-ide-opt-in.md) · enhancement, ide, reed, batten
   - Default to CLI/tmux and start VS Code only on request; alternatives: pop-up terminals or tmux windows
1. **Ship lyx as an installable Claude Code plugin** — [`plugin-packaging`](design-plugin-packaging.md) · enhancement
   - Package lyx and its skills as one installable plugin
1. **Burler picks the model per review round** — [`burler-round-models`](design-burler-round-models.md) · enhancement, burler
   - Round 1, round 2 and later rounds may each use a different base model, not only a different provider
1. **Codex as a second agent engine** — [`codex-engine`](design-codex-engine.md) · enhancement, shuttle
   - If lyx gets a second engine, Codex is more relevant than Gemini
1. **Fabric's ordinary-monorepo verb surface** — [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) · enhancement, fabric
   - Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame
1. **Talk to the orch through Slack** — [`reed-slack-relay`](design-reed-slack-relay.md) · enhancement, reed
   - A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed
1. **Agents run with a deliberate tool and permission setup that never blocks the flow** — [`agent-permission-setup`](design-agent-permission-setup.md) · enhancement, shuttle
   - A correct deny/allow setup for every lyx-spawned role, so no agent stops on a tool or command it needs
1. **Burler as two parallel agents: a review-only reviewer and a fixer that validates** — [`burler-split`](design-burler-split.md) · enhancement, burler
   - B orients, forks its lenses and writes the report only; A, stronger, orients in parallel, then fixes and validates B findings with a grain of salt
1. **Test tiers: unit, git, tmux and Claude, with Claude opt-in only** — [`test-tiers`](design-test-tiers.md) · enhancement
   - Four tiers by what a test needs and costs; only an explicit request runs the Claude tier; tier names still open
1. **Fabric exports without warp/weft in their names** — [`fabric-api-vocabulary`](design-fabric-api-vocabulary.md) · enhancement, fabric
   - 13 fabric and hubforge exports name warp/weft, so callers outside fabric must too; rename them and drop the scan exemption
1. **raddle: codeguide's woven-in successor** — [`raddle`](design-raddle.md) · enhancement, loom
   - Docs regenerated once at merge time over the plan's diff, folded into Finalize's contract; the geometry and the parallel-regeneration design exist
1. **webster: worktree-per-card parallel execution** — [`webster-parallel-execution`](design-webster-parallel-execution.md) · enhancement, webster
   - Give each DAG-independent group its own fabric-spawned worktree so concurrent cards stop sharing one git index; parked until measured evidence says it pays
1. **reed: cross-worktree columns, own-window anchoring, per-window attach** — [`reed-multi-window`](design-reed-multi-window.md) · enhancement, reed
   - Three related ideas for reed beyond one pane per strand and one window per worktree
1. **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history** — [`warp-visibility`](design-warp-visibility.md) · enhancement, ide, fabric
   - Expose CLAUDE.local.md by symlink so nothing lyx-related shows in the Fabric repo's history; the constraints half is out of date
1. **Baseline producer and numerical-drift gate** — [`baseline-drift-gate`](design-baseline-drift-gate.md) · enhancement, webster, loom
   - Capture a before-state so a failure the task caused can be told from a pre-existing one; in numerical repos the failure to catch is drift
1. **config: repo-wide default plus per-worktree override** — [`config-overlay`](design-config-overlay.md) · enhancement, config
   - A repo-wide default layer read from _board with each worktree's own file as an override, the way millhouse's config.local.yaml works
1. **fabric: two-sided reset-to-SHA verb** — [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) · enhancement, fabric
   - The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides
1. **finalize: the discrepancy-document conflict shape** — [`finalize-discrepancy-doc`](design-finalize-discrepancy-doc.md) · enhancement, fabric, loom
   - Divergences that cannot be a git conflict get a precomputed document for the resolving agent instead of markers
1. **shedrecipe: capability declaration instead of manual seam-threading** — [`shedrecipe-capabilities`](design-shedrecipe-capabilities.md) · enhancement, shed
   - A producer declares what it needs and the registry wires it, instead of a hand-threaded Env passthrough field per layer
1. **doctor: a lyx diagnostics command** — [`doctor`](design-doctor.md) · enhancement
   - lyx doctor checks the _lyx layout, config parse, board reachability and stale locks
1. **session sync: Claude transcripts across machines** — [`session-sync`](design-session-sync.md) · enhancement, shuttle
   - Copy Claude .jsonl transcripts across machines so --resume works elsewhere
1. **Interactive board view: a regenerated HTML file and a phone-friendly README** — [`board-viewer`](design-board-viewer.md) · enhancement, board, ide
   - board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app
1. **A parked label hides dormant notes from the default view** — [`board-parked`](design-board-parked.md) · enhancement, board
   - Notes nobody means to act on soon get parked: kept, hidden by default; curated at each intake
1. **lyx seeds the prime VS Code setup: title, orch start task, keybindings** — [`ide-prime-setup`](design-ide-prime-setup.md) · enhancement, ide, orch
   - LY: prime/LY: <slug> titles, prime opens with reed up+attach only, orch start and open board as tasks with lyx keybindings, no launch.json
