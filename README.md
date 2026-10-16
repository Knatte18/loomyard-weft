# Board

Rendered 2026-10-16 08:56 UTC; every age below is as of then.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks

Concrete and claimable; only a task can run.

### Running

Held by a run; its scope is locked until the run ends.

#### Normal priority

| # | Task | Slug | Recipe | At | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | **reed add defaults to pane-era placement and splits the selvage shell window**<br>• Replace the obsolete below-parent default and sweep for other pane-era leftovers | [`reed-add-parentless-placement`](design-reed-add-parentless-placement.md) | loom | Plan-Burler-Review | 1h | 1h | reed |
| 2 | **Orch role stencil fixes from the stencil intent audit**<br>• Arbiter as the orch filter with a fault fallback, no peer comparing, ORCHESTRATOR.md for repo-specific orch rules, owner of a small PR-Gate defect | [`orch-role-stencil-fixes`](design-orch-role-stencil-fixes.md) | loom | Plan-Write | 1h | 34m | stencil, orch, loom, config |
| 3 | **Loom stencil fixes from the stencil intent audit, verify tiers from hub config**<br>• Rejection findings are a reviewer claim, never the operator; driver, advisor, plan and rework wording | [`loom-stencil-wording`](design-loom-stencil-wording.md) | loom | Plan-Write | 1h | 31m | stencil, loom, shed, config, webster |
| 4 | **Creel gets its own sandbox suite**<br>• Move S9 and C1-C13 out of the shuttle sandbox suite into SANDBOX-CREEL-SUITE.md | [`creel-sandbox-suite`](design-creel-sandbox-suite.md) | loom | Plan-Burler-Review | 24m | 24m | creel, shuttle |

### Ready

Waits on nothing open; can start now.

_None._

### Layer A

Waits only on Running or Ready entries.

#### Normal priority

| # | Task | Slug | Recipe | After | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | **Webster fork and pattern-directive stencil fixes from the stencil intent audit**<br>• Structure-aware rename tool from hub config, departures reach the Master via the digest, spawn wording, no inline pack, declared PATTERN changes | [`webster-fork-stencil-fixes`](design-webster-fork-stencil-fixes.md) | loom | `loom-stencil-wording` | 1h | 30m | stencil, webster, burler, loom |

## Notes

Not tasks: ideas and observations, merged into a task when one is promoted.

### Audits

#### High priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Audit every stencil for design assumptions the operator never meant**<br>• List each claim a stencil makes about how its role works, so the operator can confirm or reject it, after the Webster master code-read bug | [`stencil-intent-audit`](design-stencil-intent-audit.md) | 20h | 34m | stencil |

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 2 | **Re-measure the test suite weight on a quiet machine**<br>• The test-suite-weight before and after numbers were taken under load; rerun the report Regenerate section on an idle machine for a fair wall-time comparison | `suite-weight-quiet-rerun` | 5d | 2d |  |
| 3 | **Prototype reed status buttons in a throwaway tmux server**<br>• A fresh agent tries the menu button, follow toggle and extra button line by hand before reed-status-buttons becomes a task | [`reed-status-buttons-prototype`](design-reed-status-buttons-prototype.md) | 23h | 23h | reed |
| 4 | **Classify Plan lens and review findings to make the Plan writer get it right first**<br>• Find what Plan lenses and Plan-Burler-Review keep catching, and stop each recurring class at the writer | [`plan-writer-finding-classes`](design-plan-writer-finding-classes.md) | 14h | 14h | loom, burler, stencil |

### Bugs

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **creel journal: a bad last line ending in a newline is skipped, not cut, and the next daemon start refuses the journal** | [`creel-journal-torn-newline-line`](design-creel-journal-torn-newline-line.md) | 0m | 0m | creel |
| 2 | **creel llm smoke test TestSmokeCreelClaudeStrand still fails with no pane capture** | [`creel-llm-smoke-idle`](design-creel-llm-smoke-idle.md) | 0m | 0m | creel, shuttle, claude |
| 3 | **reed watchdog outlives lyx reed down and respawns on a redeployed binary** | [`reed-watchdog-outlives-down`](design-reed-watchdog-outlives-down.md) | 0m | 0m | reed |
| 4 | **strand pane resolves bare lyx to the production binary while LYX_BIN names the dev build** | [`strand-pane-lyx-path`](design-strand-pane-lyx-path.md) | 0m | 0m | reed, shuttle |
| 5 | **lyx creel read to an unwritable stdout exits 1 with nothing on stderr** | [`creel-read-silent-write-fail`](design-creel-read-silent-write-fail.md) | 0m | 0m | creel |
| 6 | **creel inject --wait prints a draft stall from 37 s earlier as if current** | [`creel-stale-stall-line`](design-creel-stale-stall-line.md) | 0m | 0m | creel |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 7 | **A hub pause leaves background forks and shells running, so hibernate never confirms**<br>• Pause interrupts turns but not background forks or shells; sessions read busy and hibernate confirmation fails | [`pause-background-work`](design-pause-background-work.md) | 1d | 1d | shuttle, webster |

### Enhancements

#### High priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Tune the Burler lens fan prompts**<br>• Revisit the parked lens fan with lens and fork prompts that overlap less, measured against solo on the fan-lab fixtures | [`fan-prompt-tuning`](design-fan-prompt-tuning.md) | 5d | 15m | burler, stencil |
| 2 | **Fork the Burler fixer from the reviewer session**<br>• The fixer re-reads the context the reviewer just read; fork it from the reviewer to reuse the cache | [`burler-fixer-fork`](design-burler-fixer-fork.md) | 1d | 15m | burler |
| 3 | **Perch review analysis: rounds, tokens and late BLOCKING findings, with proposals** | [`perch-review-analysis`](design-perch-review-analysis.md) | 20m | 15m | burler, loom, shed |

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 4 | **Codex CLI as a second shuttle engine: hands-on spike, then codexengine**<br>• Codex CLI as a second engine to offload Claude tokens: short hands-on tests first, then a codexengine for single-session roles | [`codex-engine`](design-codex-engine.md) | 12d | 22h | shuttle, codex |
| 5 | **finalize: the discrepancy-document conflict shape**<br>• Divergences that cannot be a git conflict get a precomputed document for the resolving agent instead of markers | [`finalize-discrepancy-doc`](design-finalize-discrepancy-doc.md) | 11d | 11d | fabric, loom |
| 6 | **shedrecipe: capability declaration instead of manual seam-threading**<br>• A producer declares what it needs and the registry wires it, instead of a hand-threaded Env passthrough field per layer | [`shedrecipe-capabilities`](design-shedrecipe-capabilities.md) | 11d |  | shed |
| 7 | **Discussion stage has no sanctioned path to land a repository edit ahead of Plan**<br>• Discussion has no sanctioned path to commit a PATTERN edit ahead of Plan | [`discussion-lands-invariant`](design-discussion-lands-invariant.md) | 10d | 10d | loom |
| 8 | **Kick-start packs from tool locations through quarry Enclose**<br>• Turn file:line locations from verify failures, lint and audits into member glyphs with quarry enclose, then into a kick-start pack | [`enclose-kickstart`](design-enclose-kickstart.md) | 8d | 20h | quarry |
| 9 | **Verification protocol for a .NET repo: slow-suite tier, numerical-baseline capture/compare, verify timeouts and logs** | [`dotnet-verify-protocol`](design-dotnet-verify-protocol.md) | 11d | 14h | loom, webster, config |
| 10 | **Run verbs in one shared namespace, not under lyx loom**<br>• lyx loom decision/status/goto/approve act on darn runs too; move recipe-neutral run verbs to one shared namespace | [`run-verbs-shared`](design-run-verbs-shared.md) | 1d | 1d | shed, loom |
| 11 | **Clearer loom model key names, one per step**<br>• Model keys like fix name the role they set, and every review segment gets its own reviewer and fixer key | [`loom-model-key-names`](design-loom-model-key-names.md) | 1d | 1d | config, loom |
| 12 | **The arbiter in shed for every shed run, batten included**<br>• The batten Go driver has a tackler but no arbiter; move the arbiter and its keys into shed beside the tackler | [`shed-arbiter-all`](design-shed-arbiter-all.md) | 1d | 1d | shed, batten, loom, config |
| 13 | **Paste cited code into a review rounds context**<br>• Reviewers edit nothing and review a fixed commit, so code pasted at round start cannot go stale; measure it against todays reviewer | [`review-inline-code`](design-review-inline-code.md) | 4d | 34m | burler, stencil |
| 14 | **Module labels from what a task touches, not only what it is about**<br>• Half the landings that changed the shuttle core lacked the shuttle label; derive or check module labels from the plan and the diff | [`labels-from-diff`](design-labels-from-diff.md) | 22h | 22h | board, loom |
| 15 | **Start one task per overlapping group at a newly-ready start**<br>• When a landing makes several overlapping tasks ready at once, batten starts one per group and names the rest in its notice | [`batten-ready-overlap`](design-batten-ready-overlap.md) | 20h | 20h | batten |
| 16 | **Rename the `lyx` binary to `lyd` (name clash with LyX)** | [`rename-binary-lyd`](design-rename-binary-lyd.md) | 2h | 2h | stencil, shuttle |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 17 | **Fabric's Windows path behaviour verified on a Windows host**<br>• Unverified after six hardening rounds; needs a Windows host, not design; take it only if Windows becomes relevant | [`fabric-windows`](design-fabric-windows.md) | 12d | 5d | fabric |
| 18 | **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history**<br>• Expose CLAUDE.local.md by symlink so nothing lyx-related shows in the Fabric repo's history; the constraints half is out of date | [`warp-visibility`](design-warp-visibility.md) | 11d |  | ide, fabric |
| 19 | **Windows verify timeout kills the whole process tree**<br>• On Windows the verifytree timeout kills only the shell, not the process tree it started | [`verify-watchdog`](design-verify-watchdog.md) | 8d | 14h | shed |

### Features

#### High priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 5d | 3d | webster |
| 2 | **Split the orch into a run driver and a conversational orch**<br>• One agent drives runs and takes escalations; the operator talks to the other, which discusses problems and updates the board. | [`orch-split`](design-orch-split.md) | 3d | 2d | orch, batten, shed, loom |
| 3 | **creel stage 2: orch and batten notices through creel**<br>• The orch watcher and batten notices and injects go through creel; the watcher keeps only its cycle logic | [`creel-notices`](design-creel-notices.md) | 22h | 21h | creel, orch, batten |
| 4 | **creel stage 3: agents message through creel, not SendMessage**<br>• Parent asks, parent review, stop reports and shuttle send go through creel, so a non-Claude engine can take a role with a parent | [`creel-agents`](design-creel-agents.md) | 22h | 21h | creel, shuttle, loom, stencil |

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 5 | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 13d | 34m | burler |
| 6 | **Orch and batten run from the operator's own long-lived worktree, not only the prime**<br>• Every orch and batten verb refuses outside the prime; let a long-lived personal worktree be an orch's home, with pairs branching from and landing into its branch | [`orch-any-worktree`](design-orch-any-worktree.md) | 12d |  | fabric, batten, orch |
| 7 | **semantic-index: find code by concept, not by literal keyword**<br>• Semantic search over docstrings and comments; doubtful today, since agents now navigate well with grep | [`semantic-index`](design-semantic-index.md) | 12d |  | quarry |
| 8 | **Ship lyx as an installable Claude Code plugin**<br>• Package lyx and its skills as one installable plugin | [`plugin-packaging`](design-plugin-packaging.md) | 12d |  | stencil |
| 9 | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | 12d | 10d | fabric |
| 10 | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 12d |  | reed |
| 11 | **raddle: codeguide's woven-in successor**<br>• Docs regenerated once at merge time over the plan's diff, folded into Finalize's contract; the geometry and the parallel-regeneration design exist | [`raddle`](design-raddle.md) | 11d |  | loom |
| 12 | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 11d | 10d | fabric |
| 13 | **doctor: a lyx diagnostics command**<br>• lyx doctor checks the _lyx layout, config parse, board reachability and stale locks | [`doctor`](design-doctor.md) | 11d | 7d | config, board |
| 14 | **session sync: Claude transcripts across machines**<br>• Copy Claude .jsonl transcripts across machines so --resume works elsewhere | [`session-sync`](design-session-sync.md) | 11d |  | shuttle |
| 15 | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 11d | 10d | board, ide, orch |
| 16 | **Spawn audit log: every process lyx starts is recorded and traceable**<br>• One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash | [`spawn-audit-log`](design-spawn-audit-log.md) | 7d | 3d | reed, shuttle, config |
| 17 | **Configurable action buttons on the reed status bar**<br>• Clickable status-bar buttons that run configured commands: VS Code, orch distill/refresh, the follow toggle and more | [`reed-status-buttons`](design-reed-status-buttons.md) | 1d | 1d | reed, config |
| 18 | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 2h | 2h | burler, shed, orch, quarry |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 19 | **Scoped build and test for C# hubs on Windows**<br>• Verify gates in a .NET hub build and test only the projects a card touches, since a scanned Windows machine makes every build slow. | [`dotnet-scoped-verify`](design-dotnet-scoped-verify.md) | 3d | 3d | shed, webster |
