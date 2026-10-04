# Board

Entries grouped by tier, then by dependency layer.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks

Concrete and claimable; only an entry here can run.

## Next Up

Planned next, but not yet concretized.

## Notes

Not tasks: ideas and observations, merged into a task when one is promoted.

### Layer A

Waits on nothing open; can start now, in parallel.

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
1. **The orch relies on driver messages, not a scratch watcher** — [`orch-no-watcher`](design-orch-no-watcher.md) · design
   - [orch][batten] The hub watches runs with a hand-written .scratch/hub/watch.sh; drivers already message their parent, so question the watcher, and close any real gap in lyx code
1. **Each batten the orch starts runs in its own tmux window** — [`batten-windows`](design-batten-windows.md) · feature
   - [orch][batten][reed][ui] Batten runs detached and invisible; a window per run makes the window list show what the orch has in flight
1. **No recovery path needs a command the agents are denied** — [`no-denied-recovery`](design-no-denied-recovery.md) · design
   - [agents][permissions][webster] Webster named git reset --hard as the way forward, the global deny list blocks it even in bypass, and the run stopped for the operator
1. **Fabric's Windows path behaviour verified on a Windows host** — [`fabric-windows`](design-fabric-windows.md) · chore
   - [fabric][windows] Unverified after six hardening rounds; needs a Windows host, not design; take it only if Windows becomes relevant
1. **semantic-index: find code by concept, not by literal keyword** — [`semantic-index`](design-semantic-index.md) · design
   - [agents][search] Semantic search over docstrings and comments; doubtful today, since agents now navigate well with grep
1. **VS Code only on request; other ways to watch a run** — [`ide-opt-in`](design-ide-opt-in.md) · design
   - [ide][batten][reed] Default to CLI/tmux and start VS Code only on request; alternatives: pop-up terminals or tmux windows
1. **Ship lyx as an installable Claude Code plugin** — [`plugin-packaging`](design-plugin-packaging.md) · feature
   - [plugins] Package lyx and its skills as one installable plugin
1. **Burler picks the model per review round** — [`burler-round-models`](design-burler-round-models.md) · design
   - [burler][models] Round 1, round 2 and later rounds may each use a different base model, not only a different provider
1. **Codex as a second agent engine** — [`codex-engine`](design-codex-engine.md) · design
   - [shuttle][engines] If lyx gets a second engine, Codex is more relevant than Gemini
1. **Fabric's ordinary-monorepo verb surface** — [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) · feature
   - [fabric][cli] Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame
1. **Talk to the orch through Slack** — [`reed-slack-relay`](design-reed-slack-relay.md) · design
   - [reed][slack] A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed
1. **Agents run with a deliberate tool and permission setup that never blocks the flow** — [`agent-permission-setup`](design-agent-permission-setup.md) · design
   - [agents][permissions] A correct deny/allow setup for every lyx-spawned role, so no agent stops on a tool or command it needs
1. **Scope added after Discussion amends the decision record** — [`post-discussion-additions`](design-post-discussion-additions.md) · design
   - [discussion][plan][orch] Additions relayed to Plan-Write were dropped by plan review because the decision record lacked them; extend the discussion instead
1. **Burler as two parallel agents: a review-only reviewer and a fixer that validates** — [`burler-split`](design-burler-split.md) · design
   - [burler][review] B orients, forks its lenses and writes the report only; A, stronger, orients in parallel, then fixes and validates B findings with a grain of salt
1. **Test tiers: unit, git, tmux and Claude, with Claude opt-in only** — [`test-tiers`](design-test-tiers.md) · chore
   - [testing][cost] Four tiers by what a test needs and costs; only an explicit request runs the Claude tier; tier names still open
1. **Fabric exports without warp/weft in their names** — [`fabric-api-vocabulary`](design-fabric-api-vocabulary.md) · chore
   - [fabric][vocabulary] 13 fabric and hubforge exports name warp/weft, so callers outside fabric must too; rename them and drop the scan exemption
1. **raddle: codeguide's woven-in successor** — [`raddle`](design-raddle.md) · design
   - [docs][finalize] Docs regenerated once at merge time over the plan's diff, folded into Finalize's contract; the geometry and the parallel-regeneration design exist
1. **webster: worktree-per-card parallel execution** — [`webster-parallel-execution`](design-webster-parallel-execution.md) · design
   - [webster][speed] Give each DAG-independent group its own fabric-spawned worktree so concurrent cards stop sharing one git index; parked until measured evidence says it pays
1. **reed: cross-worktree columns, own-window anchoring, per-window attach** — [`reed-multi-window`](design-reed-multi-window.md) · design
   - [reed][ui] Three related ideas for reed beyond one pane per strand and one window per worktree
1. **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history** — [`warp-visibility`](design-warp-visibility.md) · design
   - [fabric][ide] Expose CLAUDE.local.md by symlink so nothing lyx-related shows in the Fabric repo's history; the constraints half is out of date
1. **Baseline producer and numerical-drift gate** — [`baseline-drift-gate`](design-baseline-drift-gate.md) · feature
   - [loom][webster][testing] Capture a before-state so a failure the task caused can be told from a pre-existing one; in numerical repos the failure to catch is drift
1. **config: repo-wide default plus per-worktree override** — [`config-overlay`](design-config-overlay.md) · feature
   - [config] A repo-wide default layer read from _board with each worktree's own file as an override, the way millhouse's config.local.yaml works
1. **fabric: two-sided reset-to-SHA verb** — [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) · feature
   - [fabric] The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides
1. **finalize: the discrepancy-document conflict shape** — [`finalize-discrepancy-doc`](design-finalize-discrepancy-doc.md) · feature
   - [finalize][fabric] Divergences that cannot be a git conflict get a precomputed document for the resolving agent instead of markers
1. **shedrecipe: capability declaration instead of manual seam-threading** — [`shedrecipe-capabilities`](design-shedrecipe-capabilities.md) · feature
   - [shed][recipes] A producer declares what it needs and the registry wires it, instead of a hand-threaded Env passthrough field per layer
1. **doctor: a lyx diagnostics command** — [`doctor`](design-doctor.md) · feature
   - [cli] lyx doctor checks the _lyx layout, config parse, board reachability and stale locks
1. **session sync: Claude transcripts across machines** — [`session-sync`](design-session-sync.md) · feature
   - [agents] Copy Claude .jsonl transcripts across machines so --resume works elsewhere
1. **Orch watcher cycles on a stale reading after /compact** — [`orch-compact-stale-reading`](design-orch-compact-stale-reading.md) · bug
   - [orch][bug] A /compact leaves the pre-compaction token reading in place, so a soft cycle fires on a near-empty context
1. **Webster blocks when a Delete target is already gone** — [`webster-delete-already-gone`](design-webster-delete-already-gone.md) · bug
   - [webster][bug] begin-batch refuses plan_drifted when an earlier batch already deleted a later card's Delete target
1. **Webster schedules a Delete before its callers are removed** — [`webster-delete-ordering`](design-webster-delete-ordering.md) · bug
   - [webster][bug] deriveEdges has no edge from a deleted symbol to the cards editing its callers
1. **Five CLI invariant exceptions in board notes, config, selfreport** — [`cli-tree-exceptions`](design-cli-tree-exceptions.md) · bug
   - [cli][bug] Fix the verbs the CLI tree walk records as exceptions and drop them from cliTreeFindings
1. **loomcli smoke tier: five bootstrap/strand tests fail on main** — [`loomcli-smoke-failures`](design-loomcli-smoke-failures.md) · bug
   - [testing][bug] Five smoke tests find no loom-status or driver strand; stale tests or a bootstrap regression
1. **Webster begin-batch spends a minute validating the plan per batch** — [`webster-batch-validation-speed`](design-webster-batch-validation-speed.md) · bug
   - [webster][quarry][perf] quarry#34 re-parses per target; fix it or validate incrementally; drop the per-batch model assertion
1. **Config changes a task makes never reach the parent** — [`config-reaches-parent`](design-config-reaches-parent.md) · bug
   - [fabric][config][landing] Live config is in weft and task weft dies at landing; carry config changes or make the hub step explicit
1. **CIRCLING escalates to the parent orchestrator, later and on recurrence only** — [`circling-to-parent`](design-circling-to-parent.md) · feature
   - [bouncer][discussion][orch] Round-2 CIRCLING on a rising finding count halted board-model for a human; judge later, on unchanged recurrence, and escalate to the parent
1. **Interactive board view: a regenerated HTML file and a phone-friendly README** — [`board-viewer`](design-board-viewer.md) · feature
   - [board][ui][ide] board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app
1. **A parked label hides dormant notes from the default view** — [`board-parked`](design-board-parked.md) · design
   - [board][undecided] Notes nobody means to act on soon get parked: kept, hidden by default; curated at each intake
1. **lyx seeds the prime VS Code setup: title, orch start task, keybindings** — [`ide-prime-setup`](design-ide-prime-setup.md) · feature
   - [ide][orch] LY: prime/LY: <slug> titles, prime opens with reed up+attach only, orch start and open board as tasks with lyx keybindings, no launch.json

### Layer B

Starts when every entry it names under After is done.

1. **A batchifier that groups small related cards into one Webster batch** — [`card-batching`](design-card-batching.md) · feature
   - [webster][batcher] Only the identity batchifier exists; a mechanical batchifier groups cards from their Create/Edit/Uses fields once cards name members as glyphs
   - **After:** `plan-glyph-precision`

## Done

Finished, awaiting `lyx board prune`.

1. **Board as one list: tasks and notes with labels, GitHub issues pulled in** — [`board-model`](design-board-model.md) · feature · done
   - [board][github] kind task|note replaces tier, labels field on both (type folds in), upsert body from file, an intake verb that turns GitHub issues into notes
