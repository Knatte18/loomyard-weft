# Board

Rendered 2026-10-12 13:25 UTC; every age below is as of then.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks

Concrete and claimable; only a task can run.

### Running

Held by a run; its scope is locked until the run ends.

#### Normal priority

| # | Task | Slug | Recipe | At | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | **Convergence report: count orch wakes, driver repairs and fresh-code fixes from existing records**<br>• A lyx report verb that counts, per run and per week, what woke the orch and what the driver repaired, from records that already exist | [`convergence-plan`](design-convergence-plan.md) | loom | Plan-Burler-Review |  | 2h | orch, shed |
| 2 | **lyx pause: stop every run at once, safely and on purpose**<br>• A hub-wide pause that interrupts every Claude session with Esc within seconds, unlike the step-boundary pauses, recorded as ordered so no failure or idle path fires | [`lyx-halt`](design-lyx-halt.md) | loom | Describe |  |  | batten, shuttle |
| 3 | **Run records tolerate newer binaries; a hung verify step is not re-run**<br>• Readers and writers of run records keep unknown fields; a verify gate stops a hung step early and never re-runs it unfixed. | [`run-robustness`](design-run-robustness.md) | loom | Webster-Burler-Fix | 2h | 2h | batten, shed, webster |

### Ready

Waits on nothing open; can start now.

_None._

### Layer A

Waits only on Running or Ready entries.

#### Normal priority

| # | Task | Slug | Recipe | After | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | **Paused runs in the overview, hold-notice end, chunked paste, operator shortcuts**<br>• Four small follow-ups: the runs overview shows pauses, the hold-notice deferral ends on a held advisor, long pastes are chunked, and shortcuts are built into lyx. | [`operator-followups`](design-operator-followups.md) | loom | `lyx-halt` | 2h | 1m | reed, shuttle, batten, orch |
| 2 | **Plan gate paths to external test packages, flaky lyxtemp sweep test, Merge-In halt with no status**<br>• Three intake bugs: validate gate-step package paths at plan time, fix the flaky concurrent-sweep test, and make a producer exit without status diagnosable. | [`intake-bugs-1012`](design-intake-bugs-1012.md) | loom | `run-robustness` | 1h | 1h | loom, webster, shed |

## Notes

Not tasks: ideas and observations, merged into a task when one is promoted.

### Bugs

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Loom halts a producer as failed with a bare 'watchdog' reason and no trace record of the kill** | [`watchdog-bare-reason`](design-watchdog-bare-reason.md) | 1h | 1h | loom, shed |
| 2 | **Weft records push refused once at Publish, passes on retry**<br>• Three runs on 2026-10-12 had the weft push refused once; find the race | [`weft-push-refused-retry`](design-weft-push-refused-retry.md) | 1h | 1h | fabric |
| 3 | **Plan card spec contradicts CLI tree invariants for a runnable group with an audience** | [`plan-cli-tree-check`](design-plan-cli-tree-check.md) | 1h | 1h | loom |
| 4 | **Shuttle marks a working agent died when its pane is collapsed**<br>• Readiness reads only the pane capture; a squeezed pane stays blank and the agent is killed at the startup timeout | [`shuttle-ready-blank-pane`](design-shuttle-ready-blank-pane.md) | 1h | 1h | shuttle |

### Enhancements

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **finalize: the discrepancy-document conflict shape**<br>• Divergences that cannot be a git conflict get a precomputed document for the resolving agent instead of markers | [`finalize-discrepancy-doc`](design-finalize-discrepancy-doc.md) |  |  | fabric, loom |
| 2 | **shedrecipe: capability declaration instead of manual seam-threading**<br>• A producer declares what it needs and the registry wires it, instead of a hand-threaded Env passthrough field per layer | [`shedrecipe-capabilities`](design-shedrecipe-capabilities.md) |  |  | shed |
| 3 | **lyx seeds the prime VS Code setup: title, orch start task, keybindings**<br>• LY: prime/LY: <slug> titles, prime opens with reed up+attach only, orch start and open board as tasks with lyx keybindings, no launch.json | [`ide-prime-setup`](design-ide-prime-setup.md) |  |  | ide, orch |
| 4 | **Discussion stage has no sanctioned path to land a repository edit ahead of Plan**<br>• Discussion has no sanctioned path to commit a PATTERN edit ahead of Plan | [`discussion-lands-invariant`](design-discussion-lands-invariant.md) |  |  | loom |
| 5 | **Kick-start packs from tool locations through quarry Enclose**<br>• Turn file:line locations from verify failures, lint and audits into member glyphs with quarry enclose, then into a kick-start pack | [`enclose-kickstart`](design-enclose-kickstart.md) |  |  | quarry |
| 6 | **Shuttle: the launch line carries the first turn**<br>• /color already goes through the verified send; the skill-load message or the prompt should ride the launch line so only later turns are typed | [`shuttle-color-verified-send`](design-shuttle-color-verified-send.md) |  |  | shuttle |
| 7 | **Config-driven file log level**<br>• logger.yaml gains a level key for the file trace, independent of -v, so a crash can be traced without a rebuild | [`logger-file-level`](design-logger-file-level.md) |  |  | config |
| 8 | **One owner for child-process lifetime in internal/proc**<br>• Fold lyx gate test's process-group, Pdeathsig and job-object setup into internal/proc beside ConfigureGroupKill | [`proc-child-lifetime`](design-proc-child-lifetime.md) |  |  |  |
| 9 | **Lens fan: master pre-reads the cited code before forking, forks get a read budget**<br>• Make the lens fan work as designed: the master reads the cited code (and judges nothing) before forking, each fork has a tool-call budget, and Discussion and Plan use their own fans | [`fan-master-preread`](design-fan-master-preread.md) |  |  | burler, stencil |
| 10 | **Re-measure the test suite weight on a quiet machine**<br>• The test-suite-weight before and after numbers were taken under load; rerun the report Regenerate section on an idle machine for a fair wall-time comparison | `suite-weight-quiet-rerun` |  |  |  |
| 11 | **Discussion chair asks advisors open questions without its options or pick** | [`chair-blind-questions`](design-chair-blind-questions.md) |  |  | loom, stencil |
| 12 | **Discussion premises cite the glyph they rest on; the gate resolves them** | [`discussion-premise-cites`](design-discussion-premise-cites.md) |  |  | loom, quarry, stencil |
| 13 | **Seats replaces SingleLLMProducer: every LLM producer is a MultiLLMProducer, zero advisors by default** | [`seats-replaces-singlellm`](design-seats-replaces-singlellm.md) |  |  | loom |
| 14 | **tokencount: guard the slug-reuse cutoff against date skew**<br>• The slug-reuse cutoff is always applied; a tag commit date skewed against transcript timestamps could silently drop sessions. | [`tokencount-cutoff-skew`](design-tokencount-cutoff-skew.md) |  |  |  |
| 15 | **Sweep tests that rely on a closed listener socket file**<br>• A forked child of a parallel test can inherit a closed listener fd and keep a stale socket answering; sweep other tests for the same race. | [`reed-socket-test-fd-sweep`](design-reed-socket-test-fd-sweep.md) |  |  | reed |
| 16 | **Measure the Webster kick-start pack with and without**<br>• Repeat the audit count (grep-for-member calls and whole-file reads per fork) on real loom runs with kickstart_pack on and off. | [`kickstart-pack-measure`](design-kickstart-pack-measure.md) |  | 44m | webster |
| 17 | **Twill for Plan: measure fewer Plan-Review rounds; wrong-parent-dir miss**<br>• After twill-plan lands: measure a lower plan_max_bounces, and cover the parent-directory miss every bench arm shared | [`twill-plan-followups`](design-twill-plan-followups.md) |  | 2h | loom |
| 18 | **Go-driven shed with a cold LLM watcher at every level**<br>• A Go loop always steps the run; an LLM watcher is spawned only when the loop stops on something it cannot pass, the same at loom and hub level | [`go-driven-shed`](design-go-driven-shed.md) | 2h | 2h | shed, batten, loom, orch |
| 19 | **No sanctioned Go formatter verb for agents that may not run rewrite scripts** | [`agent-gofmt-verb`](design-agent-gofmt-verb.md) | 1h | 1h | shuttle |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 20 | **Fabric's Windows path behaviour verified on a Windows host**<br>• Unverified after six hardening rounds; needs a Windows host, not design; take it only if Windows becomes relevant | [`fabric-windows`](design-fabric-windows.md) |  |  | fabric |
| 21 | **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history**<br>• Expose CLAUDE.local.md by symlink so nothing lyx-related shows in the Fabric repo's history; the constraints half is out of date | [`warp-visibility`](design-warp-visibility.md) |  |  | ide, fabric |
| 22 | **Windows verify timeout kills the whole process tree**<br>• On Windows the verifytree timeout kills only the shell, not the process tree it started | [`verify-watchdog`](design-verify-watchdog.md) |  | 2h | shed |
| 23 | **Bench the chair-and-advisors Discussion against the solo writer before switching it on**<br>• Same tasks through both producers, compare review rounds, findings, tokens and time; the operator watches a live run before it goes on | [`discussion-multi-llm-bench`](design-discussion-multi-llm-bench.md) |  |  | loom, stencil |
| 24 | **Discussion advisors: operator-named areas per advisor on top of the landed all-advisors default**<br>• Each discussion_advisors entry takes an optional area (a lens, as in review fans); without one the advisor answers every question | [`discussion-advisor-mode`](design-discussion-advisor-mode.md) |  |  | loom, stencil, config |
| 25 | **Fabric audit: plain readers accept unquoted globs and any flag**<br>• readOnlyCommands readers (cat, grep -r) accept unquoted globs and arbitrary flags | [`audit-reader-globs`](design-audit-reader-globs.md) |  |  | webster, fabric |

### Features

#### High priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) |  |  | webster |
| 2 | **Split the orch into a run driver and a conversational orch**<br>• One agent drives runs and takes escalations; the operator talks to the other, which discusses problems and updates the board. | [`orch-split`](design-orch-split.md) |  |  | orch, batten |

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 3 | **creel: the one way to reach an agent**<br>• Post (pulled mail with reminders), inject and interrupt to any agent by name, through one daemon per hub; built standalone first, then wired in two stages | [`creel`](design-creel.md) |  | 2h | reed |
| 4 | **Kick-start pack for Burler: pre-resolved glyph spans before the explore phase**<br>• Burler side of the kick-start pack (#228); the Webster side is in task plan-glyph-tuning | [`kick-start-pack`](design-kick-start-pack.md) |  |  | burler, quarry |
| 5 | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) |  |  | burler |
| 6 | **Orch and batten run from the operator's own long-lived worktree, not only the prime**<br>• Every orch and batten verb refuses outside the prime; let a long-lived personal worktree be an orch's home, with pairs branching from and landing into its branch | [`orch-any-worktree`](design-orch-any-worktree.md) |  |  | fabric, batten, orch |
| 7 | **semantic-index: find code by concept, not by literal keyword**<br>• Semantic search over docstrings and comments; doubtful today, since agents now navigate well with grep | [`semantic-index`](design-semantic-index.md) |  |  | quarry |
| 8 | **Ship lyx as an installable Claude Code plugin**<br>• Package lyx and its skills as one installable plugin | [`plugin-packaging`](design-plugin-packaging.md) |  |  | stencil |
| 9 | **Codex as a second agent engine**<br>• If lyx gets a second engine, Codex is more relevant than Gemini | [`codex-engine`](design-codex-engine.md) |  |  | shuttle |
| 10 | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) |  |  | fabric |
| 11 | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) |  |  | reed |
| 12 | **raddle: codeguide's woven-in successor**<br>• Docs regenerated once at merge time over the plan's diff, folded into Finalize's contract; the geometry and the parallel-regeneration design exist | [`raddle`](design-raddle.md) |  |  | loom |
| 13 | **webster: worktree-per-card parallel execution**<br>• Give each DAG-independent group its own fabric-spawned worktree so concurrent cards stop sharing one git index; parked until measured evidence says it pays | [`webster-parallel-execution`](design-webster-parallel-execution.md) |  |  | webster |
| 14 | **reed: cross-worktree columns, own-window anchoring, per-window attach**<br>• Three related ideas for reed beyond one pane per strand and one window per worktree | [`reed-multi-window`](design-reed-multi-window.md) |  |  | reed |
| 15 | **Baseline producer and numerical-drift gate**<br>• Capture a before-state so a failure the task caused can be told from a pre-existing one; in numerical repos the failure to catch is drift | [`baseline-drift-gate`](design-baseline-drift-gate.md) |  |  | webster, loom |
| 16 | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) |  |  | fabric |
| 17 | **doctor: a lyx diagnostics command**<br>• lyx doctor checks the _lyx layout, config parse, board reachability and stale locks | [`doctor`](design-doctor.md) |  |  | config, board |
| 18 | **session sync: Claude transcripts across machines**<br>• Copy Claude .jsonl transcripts across machines so --resume works elsewhere | [`session-sync`](design-session-sync.md) |  |  | shuttle |
| 19 | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) |  | 51m | board, ide |
| 20 | **Open an extra named agent strand through shuttle**<br>• Open a named Claude session in a strand with no task, through shuttle, so it keeps its name; for extra strands in the prime | [`extra-agent-strand`](design-extra-agent-strand.md) |  |  | shuttle |
| 21 | **Who in a running task receives an instruction**<br>• Driver forwarding, an alias for the active strand kept by loom, or an on-call LLM per run: open for a brainstorm | [`run-message-routing`](design-run-message-routing.md) |  |  | loom |
| 22 | **Spawn audit log: every process lyx starts is recorded and traceable**<br>• One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash | [`spawn-audit-log`](design-spawn-audit-log.md) |  |  | reed, shuttle |
| 23 | **lyx orch start --continue resumes the last orch session**<br>• lyx orch records its session id so a reboot resumes the same orch conversation instead of a fresh one. | [`orch-continue`](design-orch-continue.md) |  |  | orch |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 24 | **Plan-Write on MultiLLMProducer: advisors gather facts and check cards**<br>• Later: Plan-Write as the chair, with an advisor that maps callers, tests and verify commands per package, and possibly one that checks each card as it is written | [`plan-multi-llm`](design-plan-multi-llm.md) |  |  | loom, stencil |
| 25 | **Scoped build and test for C# hubs on Windows**<br>• Verify gates in a .NET hub build and test only the projects a card touches, since a scanned Windows machine makes every build slow. | [`dotnet-scoped-verify`](design-dotnet-scoped-verify.md) | 2h | 2h | shed, webster |
