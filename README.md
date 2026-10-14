# Board

Rendered 2026-10-14 14:57 UTC; every age below is as of then.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks

Concrete and claimable; only a task can run.

### Running

Held by a run; its scope is locked until the run ends.

#### Normal priority

| # | Task | Slug | Recipe | At | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | **tender: hub-wide pause, resume, revive and stale-binary restart**<br>• A tender module holds hub-wide pause, resume, revive and stale-binary restart, and a pause no longer consumes a call deadline | [`tender`](design-tender.md) | loom | Webster | 1d | 1d | shuttle, reed, batten, shed, orch, loom |
| 2 | **Darn: a decision record for mid-run scope corrections, and PR-Align**<br>• Darn has no decision record, so a mid-run scope correction never reaches its review; and no PR-Align row, so every darn PR waits for the orch at PR-Gate | [`claimed-scope-edit`](design-claimed-scope-edit.md) | loom | Discussion-Burler-Fix | 1d | 1d | board, loom, burler |
| 3 | **Twill: spawn lens forks after the files are written**<br>• Discussion-Write and Plan-Write spawn their lens forks once the files are written and checked; forks review what they inherited, no ready-marker wait | [`twill-spawn-after`](design-twill-spawn-after.md) | darn | Merge-In | 21m | 16m | loom, stencil |

### Ready

Waits on nothing open; can start now.

_None._

## Notes

Not tasks: ideas and observations, merged into a task when one is promoted.

### Bugs

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Plan does not surface tests that pin a method set a card extends** | [`plan-pinned-tests`](design-plan-pinned-tests.md) | 1d | 1d | loom |
| 2 | **tokencount under-counts fork output tokens**<br>• tokencount keeps the first transcript line per message id, which in subagent transcripts carries partial output_tokens; fork costs come out low | [`tokencount-fork-output`](design-tokencount-fork-output.md) | 1d | 1d |  |

### Enhancements

#### High priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Model downgrade per role**<br>• Bench cheaper models (Sonnet 5.5, Haiku 5.5) per role by replaying older runs; per-round models and fan vs solo | [`model-downgrade`](design-model-downgrade.md) | 1d | 1d | burler, loom, webster, shuttle |

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 2 | **Codex CLI as a second shuttle engine: hands-on spike, then codexengine**<br>• Codex CLI as a second engine to offload Claude tokens: short hands-on tests first, then a codexengine for single-session roles | [`codex-engine`](design-codex-engine.md) | 10d | 1d | shuttle |
| 3 | **finalize: the discrepancy-document conflict shape**<br>• Divergences that cannot be a git conflict get a precomputed document for the resolving agent instead of markers | [`finalize-discrepancy-doc`](design-finalize-discrepancy-doc.md) | 10d | 9d | fabric, loom |
| 4 | **shedrecipe: capability declaration instead of manual seam-threading**<br>• A producer declares what it needs and the registry wires it, instead of a hand-threaded Env passthrough field per layer | [`shedrecipe-capabilities`](design-shedrecipe-capabilities.md) | 10d |  | shed |
| 5 | **Discussion stage has no sanctioned path to land a repository edit ahead of Plan**<br>• Discussion has no sanctioned path to commit a PATTERN edit ahead of Plan | [`discussion-lands-invariant`](design-discussion-lands-invariant.md) | 8d | 8d | loom |
| 6 | **Kick-start packs from tool locations through quarry Enclose**<br>• Turn file:line locations from verify failures, lint and audits into member glyphs with quarry enclose, then into a kick-start pack | [`enclose-kickstart`](design-enclose-kickstart.md) | 6d | 3d | quarry |
| 7 | **Lens fan: master pre-reads the cited code before forking, forks get a read budget**<br>• Make the lens fan work as designed: the master reads the cited code (and judges nothing) before forking, each fork has a tool-call budget, and Discussion and Plan use their own fans | [`fan-master-preread`](design-fan-master-preread.md) | 4d | 3d | burler, stencil |
| 8 | **Discussion premises cite the glyph they rest on; the gate resolves them** | [`discussion-premise-cites`](design-discussion-premise-cites.md) | 3d | 3d | loom, quarry, stencil |
| 9 | **Seats replaces SingleLLMProducer: every LLM producer is a MultiLLMProducer, zero advisors by default** | [`seats-replaces-singlellm`](design-seats-replaces-singlellm.md) | 3d | 2d | loom |
| 10 | **Twill for Plan: measure fewer Plan-Review rounds; wrong-parent-dir miss**<br>• After twill-plan lands: measure a lower plan_max_bounces, and cover the parent-directory miss every bench arm shared | [`twill-plan-followups`](design-twill-plan-followups.md) | 2d | 1d | loom |
| 11 | **Arbiter spawns only on stops that need it**<br>• Five arbiter spawns on 2026-10-12, none needed: narrow the triggers to the hard review cap, circling, and blocked or error stops with no computed way forward. | [`arbiter-when`](design-arbiter-when.md) | 2d | 1d | loom, shed |
| 12 | **lyx report timing: phase minutes, parent waits and agent spawns from run records**<br>• A mechanical per-run and per-day time report beside lyx report convergence, including stops by who settled them. | [`report-timing`](design-report-timing.md) | 1d | 1d | shed, orch |
| 13 | **Make shuttle and lyx neutral to the LLM provider**<br>• Engine registry, engine in run.json, no Claude names in the core, usage readers behind the engine | [`llm-agnostic-engine`](design-llm-agnostic-engine.md) | 1d | 1d | shuttle, config, stencil |
| 14 | **Tune the Burler lens fan prompts**<br>• Revisit the parked lens fan with lens and fork prompts that overlap less, measured against solo on the fan-lab fixtures | [`fan-prompt-tuning`](design-fan-prompt-tuning.md) | 1d | 1d | burler, stencil |
| 15 | **Seed the VS Code config of each pair when it is made**<br>• Write the title-bar color and .vscode config that lyx ide spawn writes when a pair is made, so code . gives the same | [`seed-vscode-config`](design-seed-vscode-config.md) | 1d | 1d | ide, batten |
| 16 | **Overview window: one combined log and checklist, no stuck for a bounce**<br>• A bounce reads stuck, finished rows show in both log and checklist, paused time counts as row time | [`overview-window-tidy`](design-overview-window-tidy.md) | 37m | 24m | reed, shed, loom |
| 17 | **Seat names: none for a lone seat, operator-chosen names for several, no chair**<br>• burler-review-chair shows chair even with one seat; let the operator name seats | [`seat-names`](design-seat-names.md) | 30m | 27m | shed, burler, reed |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 18 | **Fabric's Windows path behaviour verified on a Windows host**<br>• Unverified after six hardening rounds; needs a Windows host, not design; take it only if Windows becomes relevant | [`fabric-windows`](design-fabric-windows.md) | 10d | 4d | fabric |
| 19 | **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history**<br>• Expose CLAUDE.local.md by symlink so nothing lyx-related shows in the Fabric repo's history; the constraints half is out of date | [`warp-visibility`](design-warp-visibility.md) | 10d |  | ide, fabric |
| 20 | **Windows verify timeout kills the whole process tree**<br>• On Windows the verifytree timeout kills only the shell, not the process tree it started | [`verify-watchdog`](design-verify-watchdog.md) | 6d | 2d | shed |
| 21 | **Bench the chair-and-advisors Discussion against the solo writer before switching it on**<br>• Same tasks through both producers, compare review rounds, findings, tokens and time; the operator watches a live run before it goes on | [`discussion-multi-llm-bench`](design-discussion-multi-llm-bench.md) | 4d | 2d | loom, stencil, config |

### Features

#### High priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 3d | 2d | webster |
| 2 | **Split the orch into a run driver and a conversational orch**<br>• One agent drives runs and takes escalations; the operator talks to the other, which discusses problems and updates the board. | [`orch-split`](design-orch-split.md) | 2d | 1d | orch, batten, shed, loom |

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 3 | **creel: the one way to reach an agent**<br>• Post (pulled mail with reminders), inject and interrupt to any agent by name, through one daemon per hub; built standalone first, then wired in two stages | [`creel`](design-creel.md) | 12d | 6d | reed |
| 4 | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 11d | 5d | burler |
| 5 | **Orch and batten run from the operator's own long-lived worktree, not only the prime**<br>• Every orch and batten verb refuses outside the prime; let a long-lived personal worktree be an orch's home, with pairs branching from and landing into its branch | [`orch-any-worktree`](design-orch-any-worktree.md) | 11d |  | fabric, batten, orch |
| 6 | **semantic-index: find code by concept, not by literal keyword**<br>• Semantic search over docstrings and comments; doubtful today, since agents now navigate well with grep | [`semantic-index`](design-semantic-index.md) | 10d |  | quarry |
| 7 | **Ship lyx as an installable Claude Code plugin**<br>• Package lyx and its skills as one installable plugin | [`plugin-packaging`](design-plugin-packaging.md) | 10d |  | stencil |
| 8 | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | 10d | 8d | fabric |
| 9 | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 10d |  | reed |
| 10 | **raddle: codeguide's woven-in successor**<br>• Docs regenerated once at merge time over the plan's diff, folded into Finalize's contract; the geometry and the parallel-regeneration design exist | [`raddle`](design-raddle.md) | 10d |  | loom |
| 11 | **Baseline producer and numerical-drift gate**<br>• Capture a before-state so a failure the task caused can be told from a pre-existing one; in numerical repos the failure to catch is drift | [`baseline-drift-gate`](design-baseline-drift-gate.md) | 10d | 8d | webster, loom |
| 12 | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 10d | 8d | fabric |
| 13 | **doctor: a lyx diagnostics command**<br>• lyx doctor checks the _lyx layout, config parse, board reachability and stale locks | [`doctor`](design-doctor.md) | 10d | 5d | config, board |
| 14 | **session sync: Claude transcripts across machines**<br>• Copy Claude .jsonl transcripts across machines so --resume works elsewhere | [`session-sync`](design-session-sync.md) | 10d |  | shuttle |
| 15 | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 9d | 8d | board, ide, orch |
| 16 | **Spawn audit log: every process lyx starts is recorded and traceable**<br>• One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash | [`spawn-audit-log`](design-spawn-audit-log.md) | 5d | 2d | reed, shuttle, config |
| 17 | **lyx orch start --continue resumes the last orch session**<br>• lyx orch records its session id so a reboot resumes the same orch conversation instead of a fresh one. | [`orch-continue`](design-orch-continue.md) | 2d |  | orch |
| 18 | **Hibernate the machine on a trigger, such as a task landing**<br>• Let the operator trigger a hibernate through lyx, for instance once a task is done | [`hibernate-trigger`](design-hibernate-trigger.md) | 42m | 41m | batten |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 19 | **Scoped build and test for C# hubs on Windows**<br>• Verify gates in a .NET hub build and test only the projects a card touches, since a scanned Windows machine makes every build slow. | [`dotnet-scoped-verify`](design-dotnet-scoped-verify.md) | 2d | 2d | shed, webster |

### Audits

#### High priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Audit glyph use in Webster, the kick-start pack first**<br>• Check from transcripts of a smaller task whether Webster forks use the kick-start pack and glyphs as intended | [`glyph-use-audit`](design-glyph-use-audit.md) | 2d | 1d | webster, burler, quarry |

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 2 | **Re-measure the test suite weight on a quiet machine**<br>• The test-suite-weight before and after numbers were taken under load; rerun the report Regenerate section on an idle machine for a fair wall-time comparison | `suite-weight-quiet-rerun` | 4d | 1d |  |
| 3 | **Twill: spawn lens forks before or after writing**<br>• A/B the twill spawn point, Discussion first: forks spawned before writing re-read the output as fresh cache writes, once per fork | [`twill-fork-timing`](design-twill-fork-timing.md) | 1d | 4m | loom, stencil |
| 4 | **Lighter test rules for small changes**<br>• Tests that pin stencil wording make small prompt changes large; pin only what must hold, measured first | [`lighter-test-rules`](design-lighter-test-rules.md) | 1d | 1d | stencil |
