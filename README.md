# Board

Rendered 2026-10-14 16:50 UTC; every age below is as of then.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks

Concrete and claimable; only a task can run.

### Running

Held by a run; its scope is locked until the run ends.

#### Normal priority

| # | Task | Slug | Recipe | At | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | **Darn: a decision record for mid-run scope corrections, and PR-Align**<br>• Darn has no decision record, so a mid-run scope correction never reaches its review; and no PR-Align row, so every darn PR waits for the orch at PR-Gate | [`claimed-scope-edit`](design-claimed-scope-edit.md) | loom | Publish | 1d | 1d | board, loom, burler |
| 2 | **Hibernate the machine on a trigger, such as a task landing**<br>• Let the operator trigger a hibernate through lyx, for instance once a task is done | [`hibernate-trigger`](design-hibernate-trigger.md) | loom | Discussion-Write | 2h | 21m | batten |
| 3 | **Overview window tidy, seat names, Seats replaces SingleLLMProducer**<br>• Overview window log/checklist and bounce naming, seat names, and one MultiLLMProducer in place of SingleLLMProducer | [`overview-seats`](design-overview-seats.md) | loom | Discussion-Write | 3d | 21m | reed, shed, loom, burler |

### Ready

Waits on nothing open; can start now.

_None._

## Notes

Not tasks: ideas and observations, merged into a task when one is promoted.

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
| 9 | **Twill for Plan: measure fewer Plan-Review rounds; wrong-parent-dir miss**<br>• After twill-plan lands: measure a lower plan_max_bounces, and cover the parent-directory miss every bench arm shared | [`twill-plan-followups`](design-twill-plan-followups.md) | 2d | 1d | loom |
| 10 | **Arbiter spawns only on stops that need it**<br>• Five arbiter spawns on 2026-10-12, none needed: narrow the triggers to the hard review cap, circling, and blocked or error stops with no computed way forward. | [`arbiter-when`](design-arbiter-when.md) | 2d | 2d | loom, shed |
| 11 | **lyx report timing: phase minutes, parent waits and agent spawns from run records**<br>• A mechanical per-run and per-day time report beside lyx report convergence, including stops by who settled them. | [`report-timing`](design-report-timing.md) | 2d | 2d | shed, orch |
| 12 | **Make shuttle and lyx neutral to the LLM provider**<br>• Engine registry, engine in run.json, no Claude names in the core, usage readers behind the engine | [`llm-agnostic-engine`](design-llm-agnostic-engine.md) | 1d | 1d | shuttle, config, stencil |
| 13 | **Tune the Burler lens fan prompts**<br>• Revisit the parked lens fan with lens and fork prompts that overlap less, measured against solo on the fan-lab fixtures | [`fan-prompt-tuning`](design-fan-prompt-tuning.md) | 1d | 1d | burler, stencil |
| 14 | **Seed the VS Code config of each pair when it is made**<br>• Write the title-bar color and .vscode config that lyx ide spawn writes when a pair is made, so code . gives the same | [`seed-vscode-config`](design-seed-vscode-config.md) | 1d | 1d | ide, batten |
| 15 | **Verification protocol for a .NET repo: slow-suite tier, numerical-baseline capture/compare, verify timeouts and logs** | [`dotnet-verify-protocol`](design-dotnet-verify-protocol.md) | 1h | 1h | loom, webster, config |
| 16 | **Darn review budget overridable per run, with a separate off switch for the perch**<br>• Darn budget keys are guard keys, so no per-run budget; make them plain keys like loom and add a guard off switch | [`darn-run-budget`](design-darn-run-budget.md) | 21m | 21m | config, shed |
| 17 | **twill fork0 bench pair twb-f2-ia**<br>• Bench pair for twill-fork-timing, Plan-Write fork0 against after, both with the critical fixer | [`twb-f2-ia`](design-twb-f2-ia.md) | 0m | 0m |  |
| 18 | **twill fork0 bench pair twb-f2-if**<br>• Bench pair for twill-fork-timing, Plan-Write fork0 against after, both with the critical fixer | [`twb-f2-if`](design-twb-f2-if.md) | 0m | 0m |  |
| 19 | **twill fork0 bench pair twb-f2-wa**<br>• Bench pair for twill-fork-timing, Plan-Write fork0 against after, both with the critical fixer | [`twb-f2-wa`](design-twb-f2-wa.md) | 0m | 0m |  |
| 20 | **twill fork0 bench pair twb-f2-wf**<br>• Bench pair for twill-fork-timing, Plan-Write fork0 against after, both with the critical fixer | [`twb-f2-wf`](design-twb-f2-wf.md) | 0m | 0m |  |
| 21 | **twill fork0 bench pair twb-f2-pa**<br>• Bench pair for twill-fork-timing, Plan-Write fork0 against after, both with the critical fixer | [`twb-f2-pa`](design-twb-f2-pa.md) | 0m | 0m |  |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 22 | **Fabric's Windows path behaviour verified on a Windows host**<br>• Unverified after six hardening rounds; needs a Windows host, not design; take it only if Windows becomes relevant | [`fabric-windows`](design-fabric-windows.md) | 10d | 4d | fabric |
| 23 | **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history**<br>• Expose CLAUDE.local.md by symlink so nothing lyx-related shows in the Fabric repo's history; the constraints half is out of date | [`warp-visibility`](design-warp-visibility.md) | 10d |  | ide, fabric |
| 24 | **Windows verify timeout kills the whole process tree**<br>• On Windows the verifytree timeout kills only the shell, not the process tree it started | [`verify-watchdog`](design-verify-watchdog.md) | 7d | 2d | shed |
| 25 | **Bench the chair-and-advisors Discussion against the solo writer before switching it on**<br>• Same tasks through both producers, compare review rounds, findings, tokens and time; the operator watches a live run before it goes on | [`discussion-multi-llm-bench`](design-discussion-multi-llm-bench.md) | 4d | 2d | loom, stencil, config |

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
| 4 | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 11d | 6d | burler |
| 5 | **Orch and batten run from the operator's own long-lived worktree, not only the prime**<br>• Every orch and batten verb refuses outside the prime; let a long-lived personal worktree be an orch's home, with pairs branching from and landing into its branch | [`orch-any-worktree`](design-orch-any-worktree.md) | 11d |  | fabric, batten, orch |
| 6 | **semantic-index: find code by concept, not by literal keyword**<br>• Semantic search over docstrings and comments; doubtful today, since agents now navigate well with grep | [`semantic-index`](design-semantic-index.md) | 10d |  | quarry |
| 7 | **Ship lyx as an installable Claude Code plugin**<br>• Package lyx and its skills as one installable plugin | [`plugin-packaging`](design-plugin-packaging.md) | 10d |  | stencil |
| 8 | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | 10d | 8d | fabric |
| 9 | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 10d |  | reed |
| 10 | **raddle: codeguide's woven-in successor**<br>• Docs regenerated once at merge time over the plan's diff, folded into Finalize's contract; the geometry and the parallel-regeneration design exist | [`raddle`](design-raddle.md) | 10d |  | loom |
| 11 | **Baseline producer and numerical-drift gate**<br>• Capture a before-state so a failure the task caused can be told from a pre-existing one; in numerical repos the failure to catch is drift | [`baseline-drift-gate`](design-baseline-drift-gate.md) | 10d | 8d | webster, loom |
| 12 | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 10d | 8d | fabric |
| 13 | **doctor: a lyx diagnostics command**<br>• lyx doctor checks the _lyx layout, config parse, board reachability and stale locks | [`doctor`](design-doctor.md) | 10d | 6d | config, board |
| 14 | **session sync: Claude transcripts across machines**<br>• Copy Claude .jsonl transcripts across machines so --resume works elsewhere | [`session-sync`](design-session-sync.md) | 10d |  | shuttle |
| 15 | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 10d | 8d | board, ide, orch |
| 16 | **Spawn audit log: every process lyx starts is recorded and traceable**<br>• One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash | [`spawn-audit-log`](design-spawn-audit-log.md) | 5d | 2d | reed, shuttle, config |
| 17 | **lyx orch start --continue resumes the last orch session**<br>• lyx orch records its session id so a reboot resumes the same orch conversation instead of a fresh one. | [`orch-continue`](design-orch-continue.md) | 3d |  | orch |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 18 | **Scoped build and test for C# hubs on Windows**<br>• Verify gates in a .NET hub build and test only the projects a card touches, since a scanned Windows machine makes every build slow. | [`dotnet-scoped-verify`](design-dotnet-scoped-verify.md) | 2d | 2d | shed, webster |

### Audits

#### High priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Audit glyph use in Webster, the kick-start pack first**<br>• Check from transcripts of a smaller task whether Webster forks use the kick-start pack and glyphs as intended | [`glyph-use-audit`](design-glyph-use-audit.md) | 3d | 1d | webster, burler, quarry |

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 2 | **Re-measure the test suite weight on a quiet machine**<br>• The test-suite-weight before and after numbers were taken under load; rerun the report Regenerate section on an idle machine for a fair wall-time comparison | `suite-weight-quiet-rerun` | 4d | 1d |  |
| 3 | **Twill: spawn lens forks before or after writing**<br>• A/B the twill spawn point, Discussion first: forks spawned before writing re-read the output as fresh cache writes, once per fork | [`twill-fork-timing`](design-twill-fork-timing.md) | 1d | 1h | loom, stencil |
| 4 | **Lighter test rules for small changes**<br>• Tests that pin stencil wording make small prompt changes large; pin only what must hold, measured first | [`lighter-test-rules`](design-lighter-test-rules.md) | 1d | 1d | stencil |
