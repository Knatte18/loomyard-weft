# Board

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks

Concrete and claimable; only a task can run.

### Layer A

Waits on nothing open; can start now, in parallel.

1. **Prune the six largest test packages** — [`test-suite-prune-core`](design-test-suite-prune-core.md) · enhancement · running · Webster
   - Stage 2b: delete redundant tests and fold near-identical ones in fabricengine, websterengine, reedengine, shuttleengine, loomcli, shedadapters
   - **Before:** `test-tiers`, `bugfix-sessions`, `bugfix-webster`, `bugfix-fabric`
1. **Prune the remaining test packages** — [`test-suite-prune-rest`](design-test-suite-prune-rest.md) · enhancement · running · Webster
   - Stage 2c: delete redundant tests and fold near-identical ones in every package outside prune-core, in parallel with it
   - **Before:** `test-tiers`, `bugfix-fabric`
1. **A spawned role loads all its skills at once, not one turn per skill** — [`skills-in-one-turn`](design-skills-in-one-turn.md) · enhancement, shuttle, stencil · awaiting · Webster-Bouncer
   - lyx types each skill as its own slash-command turn, each ending in chatter; one typed message loads them all through the Skill tool in one turn
   - **Before:** `bugfix-sessions`

### Layer B

Starts when every entry it names under After is done.

1. **Which gate runs which test tier, after-state and budget guard** — [`test-tiers`](design-test-tiers.md) · enhancement, loom, webster
   - Stage 3: review rounds run the impacted set, full plan verify only at Webster gate and Publish/Finalize; after-state recorded; per-package test budget guard
   - **After:** `test-suite-prune-core`, `test-suite-prune-rest`
1. **Bugfix: session lifecycle, notices and run halts** — [`bugfix-sessions`](design-bugfix-sessions.md) · bug, shuttle, reed, loom, orch, burler
   - Bugfix only: prompt delivery, stale producer sessions, false crash notes, watcher restart, Burler YAML repair, no client switch from agents
   - **After:** `skills-in-one-turn`, `test-suite-prune-core`
1. **Bugfix: Webster and plan validity** — [`bugfix-webster`](design-bugfix-webster.md) · bug, webster, loom, quarry
   - Bugfix only: Plan-Gate refuses backward dependencies, Delete of a gone target, background-shell wait, slow batch validation
   - **After:** `test-suite-prune-core`
1. **Bugfix: fabric and config propagation** — [`bugfix-fabric`](design-bugfix-fabric.md) · bug, fabric, config, reed
   - Bugfix only: a weft without .weft/ drops LYX_PARENT, and task config changes never reach the parent
   - **After:** `test-suite-prune-core`, `test-suite-prune-rest`

## Notes

Not tasks: ideas and observations, merged into a task when one is promoted.

### Bugs

1. **A valid plan runs in card order: Plan-Gate refuses backward dependencies, Webster never reorders** — [`webster-delete-ordering`](design-webster-delete-ordering.md) · bug, webster
   - Card N never depends on a later card; Plan-Gate enforces it and Webster drops its own sequencing
1. **The orch resumes runs without commands that take over the operator's terminal** — [`orch-resume-without-attach`](design-orch-resume-without-attach.md) · bug, orch, loom, reed
   - lyx loom start from the orch switches the operator's tmux client; the orch needs a side-effect-free resume verb

### Enhancements

1. **Convergence: measure hub and driver interventions, fix at the source, hardening freezes** — [`convergence-plan`](design-convergence-plan.md) · enhancement
   - Plan for lyx to converge rather than accumulate bugs: per-run intervention log, real-wiring tests, feature freezes
1. **Plan cards name the members they change as glyphs, after lyx's use of glyphs is tested systematically** — [`plan-glyph-precision`](design-plan-glyph-precision.md) · enhancement, loom, quarry
   - Plan-Write lists file paths instead of member glyphs because its stencil shows and allows it, and handle-unreferenced forces invented glyphs; test lyx's use of quarry glyphs systematically first
1. **Prompt-cache TTL per agent role** — [`cache-ttl-per-role`](design-cache-ttl-per-role.md) · enhancement, shuttle
   - Parked: workers on 5m cache TTL, waiting sessions on 1h — operator deciding
1. **Parent-review gate: wait bound off the boundary, no mid-turn sends, live smoke** — [`parent-review-hardening`](design-parent-review-hardening.md) · enhancement, orch
   - Close the cases the first parent-review gate left open: expiry while the writer is mid-turn, sends into a running turn after restart, poll cost, and one live smoke run
1. **Reopen the prime VS Code window with one short command** — [`ide-reopen-prime`](design-ide-reopen-prime.md) · enhancement, ide
   - A snap refresh left the prime window wrong; lyx ide spawn needs the slug and fails from the hub root
1. **Fabric's Windows path behaviour verified on a Windows host** — [`fabric-windows`](design-fabric-windows.md) · enhancement, fabric
   - Unverified after six hardening rounds; needs a Windows host, not design; take it only if Windows becomes relevant
1. **VS Code only on request; other ways to watch a run** — [`ide-opt-in`](design-ide-opt-in.md) · enhancement, ide, reed, batten
   - Default to CLI/tmux and start VS Code only on request; alternatives: pop-up terminals or tmux windows
1. **Agents run with a deliberate tool and permission setup that never blocks the flow** — [`agent-permission-setup`](design-agent-permission-setup.md) · enhancement, shuttle
   - A correct deny/allow setup for every lyx-spawned role, so no agent stops on a tool or command it needs
1. **Burler as two parallel agents: a review-only reviewer and a fixer that validates** — [`burler-split`](design-burler-split.md) · enhancement, burler
   - B orients, forks its lenses and writes the report only; A, stronger, orients in parallel, then fixes and validates B findings with a grain of salt
1. **Fabric exports without warp/weft in their names** — [`fabric-api-vocabulary`](design-fabric-api-vocabulary.md) · enhancement, fabric
   - 13 fabric and hubforge exports name warp/weft, so callers outside fabric must too; rename them and drop the scan exemption
1. **webster: worktree-per-card parallel execution** — [`webster-parallel-execution`](design-webster-parallel-execution.md) · enhancement, webster
   - Give each DAG-independent group its own fabric-spawned worktree so concurrent cards stop sharing one git index; parked until measured evidence says it pays
1. **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history** — [`warp-visibility`](design-warp-visibility.md) · enhancement, ide, fabric
   - Expose CLAUDE.local.md by symlink so nothing lyx-related shows in the Fabric repo's history; the constraints half is out of date
1. **finalize: the discrepancy-document conflict shape** — [`finalize-discrepancy-doc`](design-finalize-discrepancy-doc.md) · enhancement, fabric, loom
   - Divergences that cannot be a git conflict get a precomputed document for the resolving agent instead of markers
1. **shedrecipe: capability declaration instead of manual seam-threading** — [`shedrecipe-capabilities`](design-shedrecipe-capabilities.md) · enhancement, shed
   - A producer declares what it needs and the registry wires it, instead of a hand-threaded Env passthrough field per layer
1. **lyx seeds the prime VS Code setup: title, orch start task, keybindings** — [`ide-prime-setup`](design-ide-prime-setup.md) · enhancement, ide, orch
   - LY: prime/LY: <slug> titles, prime opens with reed up+attach only, orch start and open board as tasks with lyx keybindings, no launch.json
1. **A running verify gate is visible in status and on screen** — [`gate-running-visible`](design-gate-running-visible.md) · enhancement, loom, reed
   - After an agent ends its turn, a Go gate can run for minutes with nothing on screen; loom status and something on screen name the gate and its runtime
1. **fabric fetches an existing remote warp and weft branch instead of creating new ones** — [`fabric-add-adopt-remote`](design-fabric-add-adopt-remote.md) · enhancement, fabric
   - A branch that exists on origin is fetched and checked out, never recreated locally; reconcile and checkout today fork a fresh weft and fabric add refuses the warp
1. **Each strand role gets its own color** — [`strand-colors`](design-strand-colors.md) · enhancement, reed, shuttle
   - Claude Code /color per strand role or loom segment, so the driver and its agents look different
1. **lyx webster reset covers the report-head and last-batch-head targets** — [`webster-reset-targets`](design-webster-reset-targets.md) · enhancement, webster
   - Three Webster recovery sites and standalone mode still instruct git reset --keep; widen the reset verb so every recovery instruction names a lyx verb
1. **reed shows strands as zoom tabs and runs as session tabs** — [`reed-status-tabs`](design-reed-status-tabs.md) · enhancement, reed
   - Stacked layout stays as VIEW; a strand button or Alt+arrows zooms one strand; runs as session tabs below; reed pins it at boot
1. **Implementers hard-wrap comments and reviews re-find it every round** — [`prose-line-breaks-in-review`](design-prose-line-breaks-in-review.md) · enhancement, webster
   - Bouncer re-finds hard-wrapped Go comments across rounds; catch it in the card gate or the implementer stencil
1. **Discussion stage has no sanctioned path to land a repository edit ahead of Plan** — [`discussion-lands-invariant`](design-discussion-lands-invariant.md) · enhancement, loom
   - Discussion has no sanctioned path to commit a PATTERN edit ahead of Plan
1. **One prompt notice channel to the orch: batten, event-driven, timestamped** — [`orch-notice-channel`](design-orch-notice-channel.md) · enhancement, batten, orch
   - Driver and batten both report every halt, and batten's 30 s poll makes its notices late and stale

### Features

1. **An MCP server over lyx exposing the verbs agents need** — [`lyx-mcp`](design-lyx-mcp.md) · feature
   - A few coarse MCP tools over lyx, chosen by measured agent use and exposed per role, so agents see typed schemas instead of guessing CLI shapes
1. **creel: a mailbox between named lyx processes** — [`creel`](design-creel.md) · feature, reed
   - Mail store and verbs on top of agent-naming's names; deferred while Claude's own SendMessage is enough
1. **Runs overview and event log in their own window in the prime session** — [`runs-overview`](design-runs-overview.md) · feature, reed, orch, batten
   - An own tmux window beside the orch: one line per run on top, a log of batten notices with attach commands below; the orch notice drops its attach command
1. **Discussion advisor: a dedicated agent answers Discussion-Write's questions live** — [`discussion-advisor`](design-discussion-advisor.md) · feature, shuttle, loom
   - A second agent spawned beside Discussion-Write studies the task and code and answers its design questions as they arise, a real two-agent dialogue
1. **Kick-start pack: pre-resolved glyph spans for implementer and Burler** — [`kick-start-pack`](design-kick-start-pack.md) · feature, webster, quarry
   - Someday (roadmap): #227, #228 — measured win, committed, not yet scheduled
1. **Orch owns the run loop: detached batten start, a lyx watch, one land command, --continue, a committed role prompt** — [`orch-owns-the-loop`](design-orch-owns-the-loop.md) · feature, batten, orch
   - Replace the hub scratch scripts and remembered procedure with lyx itself, so an orch session runs the loop from its committed role prompt alone and resumes after a reboot
1. **Hardener: an autonomous behavior-review campaign that replaces crucible** — [`hardener`](design-hardener.md) · feature
   - Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges
1. **A batchifier that groups small related cards into one Webster batch** — [`card-batching`](design-card-batching.md) · feature, webster
   - Only the identity batchifier exists; a mechanical batchifier groups cards from their Create/Edit/Uses fields once cards name members as glyphs
1. **Orch and batten run from the operator's own long-lived worktree, not only the prime** — [`orch-any-worktree`](design-orch-any-worktree.md) · feature, fabric, batten, orch
   - Every orch and batten verb refuses outside the prime; let a long-lived personal worktree be an orch's home, with pairs branching from and landing into its branch
1. **lyx ide open: launch VS Code without writing anything** — [`ide-open-verb`](design-ide-open-verb.md) · feature, ide, batten
   - spawn configures and launches at once and overwrites a driven pair's attach chain; add a write-nothing open verb
1. **Pairs as folders in the prime's hub workspace** — [`ide-workspace-pairs`](design-ide-workspace-pairs.md) · feature, ide, fabric
   - Idea, undecided: fabric adds and removes pairs in the prime workspace's folders; _portals stays
1. **Each batten the orch starts runs in its own tmux window** — [`batten-windows`](design-batten-windows.md) · feature, reed, batten, orch
   - Batten runs detached and invisible; a window per run makes the window list show what the orch has in flight
1. **semantic-index: find code by concept, not by literal keyword** — [`semantic-index`](design-semantic-index.md) · feature
   - Semantic search over docstrings and comments; doubtful today, since agents now navigate well with grep
1. **Ship lyx as an installable Claude Code plugin** — [`plugin-packaging`](design-plugin-packaging.md) · feature
   - Package lyx and its skills as one installable plugin
1. **Burler picks the model per review round** — [`burler-round-models`](design-burler-round-models.md) · feature, burler
   - Round 1, round 2 and later rounds may each use a different base model, not only a different provider
1. **Codex as a second agent engine** — [`codex-engine`](design-codex-engine.md) · feature, shuttle
   - If lyx gets a second engine, Codex is more relevant than Gemini
1. **Fabric's ordinary-monorepo verb surface** — [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) · feature, fabric
   - Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame
1. **Talk to the orch through Slack** — [`reed-slack-relay`](design-reed-slack-relay.md) · feature, reed
   - A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed
1. **raddle: codeguide's woven-in successor** — [`raddle`](design-raddle.md) · feature, loom
   - Docs regenerated once at merge time over the plan's diff, folded into Finalize's contract; the geometry and the parallel-regeneration design exist
1. **reed: cross-worktree columns, own-window anchoring, per-window attach** — [`reed-multi-window`](design-reed-multi-window.md) · feature, reed
   - Three related ideas for reed beyond one pane per strand and one window per worktree
1. **Baseline producer and numerical-drift gate** — [`baseline-drift-gate`](design-baseline-drift-gate.md) · feature, webster, loom
   - Capture a before-state so a failure the task caused can be told from a pre-existing one; in numerical repos the failure to catch is drift
1. **config: repo-wide default plus per-worktree override** — [`config-overlay`](design-config-overlay.md) · feature, config
   - A repo-wide default layer read from _board with each worktree's own file as an override, the way millhouse's config.local.yaml works
1. **fabric: two-sided reset-to-SHA verb** — [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) · feature, fabric
   - The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides
1. **doctor: a lyx diagnostics command** — [`doctor`](design-doctor.md) · feature
   - lyx doctor checks the _lyx layout, config parse, board reachability and stale locks
1. **session sync: Claude transcripts across machines** — [`session-sync`](design-session-sync.md) · feature, shuttle
   - Copy Claude .jsonl transcripts across machines so --resume works elsewhere
1. **Interactive board view: a regenerated HTML file and a phone-friendly README** — [`board-viewer`](design-board-viewer.md) · feature, board, ide
   - board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app
1. **Shortcut commands built into lyx from one committed file** — [`operator-shortcuts`](design-operator-shortcuts.md) · feature, orch, reed
   - An embedded shortcut file, git-alias style: attach for reed attach, and one shortcut that starts the orch and attaches in the prime

## Done

Finished, awaiting `lyx board prune`.

1. **lyx files bug issues from Go code; only reflection agents should** — [`selfreport-issues-say-nothing`](design-selfreport-issues-say-nothing.md) · bug, selfreport, loom · done
   - Tier 1 anomaly issues carry only keys and blank fields; drop Go-side filing and feed anomalies to Friction-Reflect instead
1. **Measure the test suite, make it fast, settle its rules** — [`test-suite-measure`](design-test-suite-measure.md) · enhancement · done
   - Stage 2a: per-test coverage tool and before-state, real-time seams and t.Parallel, tmux socket leak, an LLM test tier, PATTERN Testing consolidated
1. **board.yaml is read per worktree though the board is one per hub** — [`board-config-per-worktree`](design-board-config-per-worktree.md) · bug, board, config · done
   - A pair with an older board.yaml re-renders README with the old types; move board.yaml beside fabric.yaml in _board
1. **Agents stop wrapping commands: stdin without a hook, a bare shed step** — [`agent-command-wrapping`](design-agent-command-wrapping.md) · bug, enhancement, shuttle, shed, stencil · done
   - Give agent shells /dev/null stdin without a per-call hook; lyx shed step prints a short envelope so the driver runs the bare call
