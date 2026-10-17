# Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)

`orchestrator-survey-ideas` · Note · feature, burler, shed, orch, quarry, report

Imported from [issue #671](https://github.com/Knatte18/loomyard/issues/671).

This issue collects features from comparable open-source agent orchestrators that LoomYard could borrow later.
I surveyed each project's README, docs and some source through the GitHub API, plus a few vendor docs pages and fresh web searches, in October 2026.
I left out anything LoomYard already has: a deterministic phase engine, glyph-validated plans, independent review with a judge and a bounce budget, warp/weft records in git, per-task worktree pairs, creel, tender, the self-cycling orch, and severity-tagged review findings with BLOCKING verdicts.
Where LoomYard already solves the same problem another way, the bullet says so, since the other project's approach may still hold something to improve.
Each bullet names its source and the LoomYard module it would touch.
Nothing here is a commitment; these are leads to triage.

## Plan validation and concurrency (quarry, Plan-Write, Batchifier, board)

- **Typed scope operations on cards** — [foremerge](https://github.com/naw103/foremerge) (`docs/conflict-detection.md`, `src/conflict.rs`).
  Each declared scope carries an operation: additive (`add`, `extend`, `modify`) or breaking (`replace`, `remove`, `rename`, `migrate`).
  Deterministic rules flag destructive-vs-additive and divergent-rewrite collisions, even when the files do not overlap.
  LoomYard already types its cards (Create/Edit/Delete/Rename/Move), keeps overlapping tasks apart through board labels, and runs the alignment gate before plan writing and before merge.
  Worth comparing: whether foremerge's operation-pair rules catch a collision the board labels and the alignment gate would miss.
- **Symbol leases between concurrent tasks** — [Fusion](https://github.com/Runfusion/Fusion) (`packages/core/src/tasks/symbol-lock-types.ts`, `file-scope-lease.ts`).
  Durable, expiring leases on symbol keys serialize tasks whose claims overlap.
  Ties are broken deterministically by priority, then age, then id.
  LoomYard covers the same need with board labels that keep overlapping tasks from running in parallel, plus the merge conflict resolver.
  Worth comparing: symbol-level granularity versus label-level, and the deterministic tie-break.
  [ORCH](https://github.com/oxgeneral/ORCH) (`src/domain/scope.ts`) does a cruder glob-prefix version.
- **Requirement-to-card traceability gate** — [Spec Kit](https://github.com/github/spec-kit) `/analyze` (`templates/commands/analyze.md`) and [Kiro](https://kiro.dev/docs/specs/correctness/).
  Spec Kit reports requirement→task coverage % and terminology drift; Kiro links each property test to a requirement and a task.
  For LoomYard: give decision-record decisions IDs and have the Plan-Write gate refuse a plan with an uncovered decision or a card that traces to none.
- **Criterion amendment ledger** — [oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode) (`skills/ralph/SKILL.md`, `docs/adr/03664-ralph-prd-criterion-amendment.md`).
  An acceptance criterion can only be relaxed through a recorded amendment that keeps the original text, reason, evidence and authority.
  For LoomYard: webster and the Webster-review judge could refuse a card whose acceptance text changed without such an entry in weft.
- **Blast-radius verdict per glyph** — [tree-sitter-analyzer](https://github.com/aimasteracc/tree-sitter-analyzer).
  Impact analysis returns SAFE/CAUTION/UNSAFE for an edit target.
  For LoomYard: a `quarry impact <glyph>` that Plan review reads, beyond the existing existence check.
- **Bugfix spec with fail-on-base test** — [Kiro bugfix specs](https://kiro.dev/docs/specs/bugfix-specs/).
  Current / Expected / Unchanged behaviour in WHEN/THEN form, each becoming a test.
  For LoomYard: a bugfix stencil for Discussion-Write, plus a Go gate that the new test fails on the base commit and passes on the tip.

## Review gates (burler, Bouncer, gate)

- **Cross-model external review with rebuttal** — [ralphex](https://github.com/umputun/ralphex) (README "Custom External Review", `pkg/config/defaults/prompts/codex.txt`).
  Any script that prints `file:line - issue` or `NO ISSUES FOUND` can review the diff.
  Claude fixes each finding or argues against it, and the rebuttal goes into the next round.
  Round 1 reviews the full branch diff; later rounds review only the fixes.
  For LoomYard: a non-Claude reviewer as a second shuttle engine inside a review segment, for independence across model families.
- **Stalemate detection** — ralphex `--review-patience=N`.
  It stops after N rounds that produce no commit and no tree change.
  For LoomYard: the Bouncer could halt a segment before it exhausts its bounce budget.
- **Review verdict bound to content** — [Fusion](https://github.com/Runfusion/Fusion) (`docs/workflow-steps.md` "Content-binding review proof", `packages/core/src/planner/plan-approval.ts`).
  An approval records a fingerprint of the plan or diff, and merge refuses a stale or unprovable approval.
  For LoomYard: stamp each Bouncer verdict in weft with the warp tree SHA and have Publish/PR-Gate refuse one that no longer matches.
  Check first whether webster's plan fingerprint already covers the plan side.
- **Incremental re-verify and "Outdated" findings** — [Traycer](https://docs.traycer.ai/extension/tasks/verification.md) and Fusion's severity-gated verdicts.
  A re-review checks only earlier findings and marks them resolved or outdated; Fusion forbids new non-blocking findings on a re-review.
  For LoomYard: a cheaper re-review round mode for burler, to cut circling.
- **Quorum of differently-angled judges** — [ccswarm](https://github.com/nwiizo/ccswarm) Sangha (`crates/ccswarm/src/workflow/sangha.rs`).
  N read-only members (planner, reviewer, QA) each emit `APPROVE|REVISE`, and the stage passes on quorum.
  For LoomYard: an optional multi-judge Bouncer for high-risk segments.
  ccswarm itself has not implemented dissent resolution yet.
- **Post-implementation convergence check** — [Spec Kit](https://github.com/github/spec-kit) `/converge` (`templates/commands/converge.md`).
  It classifies gaps as missing, partial, contradicts or **unrequested**, ignores checkbox claims as evidence, and appends a remediation phase rather than free-form fixes.
  For LoomYard: an "unrequested" lens in Webster review against the decision record, to catch scope creep.
- **Risk tier by changed paths** — [yylo](https://github.com/yylo-dev/yylo) (`src/templates/config/risk-policy.json`).
  Globs over auth, migrations, lockfiles and CI pick the reviewer count and whether the full suite runs.
  For LoomYard: choose a segment's review depth and `lyx gate test` scope from the diff.
  yylo has since marked this queue as retired (`docs/lifecycle-simplification-migration.md`), so treat it as an idea only.
- **Pre-existing failure baseline** — oh-my-claudecode `ralph verify --write-baseline`.
  Gates fail only on new failures; baseline failures become warnings.
  For LoomYard: `lyx gate test` and the webster verify gate could record a base-commit baseline, so implementers stop chasing failures that were already red.
  Counterpoint: LoomYard holds that main is always green, so a baseline would legitimize a red main, and it costs an extra full suite run before work starts.
- **Typed completion signals** — [Bernstein](https://github.com/sipyourdrink-ltd/bernstein) janitor (`src/bernstein/core/quality/janitor.py`).
  A task declares `path_exists`, `test_passes` or `file_contains`; an LLM judge runs only once those pass.
  For LoomYard: cards could declare done-conditions that `record-batch` checks in Go.

## Engine, recipes and governance (shed, tender, orch, board)

- **Hard token and cost budgets** — Fusion per-task budgets (`docs/settings-reference.md#per-task-token-budget`) and [Conductor](https://github.com/microsoft/conductor) `limits.budget_usd` with `budget_mode: audit|enforce` (`docs/workflow-syntax.md`).
  A soft cap alerts once, and a hard cap pauses with a named reason.
  Conductor recommends running in audit mode first.
  For LoomYard: weft already records token usage per round; shed or tender could enforce a per-run cap, starting in audit mode.
- **Cross-run quarantine** — Bernstein (`src/bernstein/core/security/quarantine.py`).
  A task that fails 3 times across runs is skipped or decomposed, and the entry expires after 7 days; host-resource failures are excused.
  For LoomYard: board plus orch, so the orch stops relaunching a doomed task.
- **Abandon as a terminal state with a closed reason list** — Bernstein (`docs/operations/abandonments.md`).
  `ABANDONED` is distinct from `FAILED`, and dependent tasks cascade to `BLOCKED_BY_ABANDON`.
  For LoomYard: shed's terminal outcomes, board dependencies and `report convergence`.
- **Fork a run from a step with a tree snapshot** — Bernstein (`docs/operations/fork-from-step.md`).
  The worktree is snapshotted to `refs/bernstein/snapshots/<run>/<step>` at each step.
  For LoomYard: `shed goto` moves the cursor but does not restore the warp tree; a per-producer ref would.
- **Warm retry guarded by a workspace hash** — Bernstein (`docs/operations/checkpointed-retries.md`).
  A retry resumes or forks the native session only if the workspace hash has not changed, and otherwise falls back to a cold restart.
  For LoomYard: webster's fork recovery.
- **Per-run pinning of prompts** — Conductor `bundle build`; Bernstein refuses to resume after its spec digest changes.
  For LoomYard: record the stencil digests a run started with in weft and have resume warn on drift, complementing `lyx stencil diff`.
- **Evidence reuse by input identity** — yylo (`risk_policy.py` `semantic_reuse_decision`).
  Earlier test or review evidence is reused only if its inputs and prompt identity are unchanged; a changed review prompt reruns only the review.
  For LoomYard: shed resume and the gate after a rework.
- **Recipe validator and non-LLM step types** — Conductor `conductor validate` and its script/set steps.
  It catches unreachable steps, bad route targets and undeclared inputs offline, and script steps route on exit code or stdout JSON.
  For LoomYard: extend the shed recipe validator with reachability and stencil output-name checks; a generic script producer could route on `lyx gate test`.
- **Mid-run guidance at the next step boundary** — Conductor `conductor guide --text`.
  For LoomYard: creel injects into a live pane today; this would queue guidance for the next producer's prompt instead.
- **Batched operator questions with recorded assumptions** — Conductor `questions` step and Spec Kit `/clarify`.
  Spec Kit asks at most 5 multiple-choice questions ranked by impact × uncertainty, each with a recommended answer, and logs answers in a dated Clarifications section.
  Conductor records unanswered questions as assumptions.
  For LoomYard: Discussion-Write's ask-the-operator protocol and the decision record.
- **Proactive rate-limit wait** — oh-my-claudecode (`src/features/rate-limit-wait/daemon.ts`).
  It reads the 5h/weekly usage windows and their reset times, then resumes blocked panes when the window resets.
  For LoomYard: tender schedules the resume instead of waiting for `revive`.
- **Standing goals and scheduled routines** — ORCH goals (`src/domain/goal.ts`, `seedAutonomousTasks`), [CAO](https://github.com/awslabs/cli-agent-orchestrator) cron flows with a gate script (`docs/flows.md`), and Fusion routines (README only).
  For LoomYard: board plus orch, for periodic triage, dependency bumps or `selfreport` sweeps.

## Worktree lifecycle hooks and safety (fabric, batten, Merge-In)

- **User lifecycle hooks** — [dmux](https://github.com/standardagents/dmux) (`src/utils/hooks.ts`, `context/HOOKS.md`).
  Executable hooks named `worktree_created`, `before_worktree_remove`, `pre_merge`, `post_merge`, `run_test` and others are looked up in a team dir (`.dmux-hooks/`), then a local dir, then a global one.
  They receive the slug, worktree path, branch and target branch as env vars.
  For LoomYard: `fabric add` and batten could run hub-configured hooks after pair creation (copy `.env`, install dependencies) and around Merge-In/landing.
  Today fabric installs only its own post-checkout drift hook.
- **Commit-time reservation guard** — [ntm](https://github.com/Dicklesworthstone/ntm) (`ntm guards check --staged`).
  A pre-commit hook refuses commits touching files another agent holds.
  For LoomYard: an enforcement point for the glyph leases above.
- **Sensitive-path deny-list before commit** — ccswarm (README "Safety").
  It blocks `.env`, `*.pem`, `id_rsa`, `credentials*` and `secrets.y*ml`.
  For LoomYard: a cheap check in `record-batch` and the fabric commit path.
- **Free a paused task's worktree** — [claude-squad](https://github.com/smtg-ai/claude-squad) (`session/instance.go`).
  Pause commits a dirty tree, removes the worktree and keeps the branch; resume recreates it.
  For LoomYard: tender/hibernate for long-parked runs; the weft sibling makes this harder.

## Agent sessions (reed, shuttle)

- **Status from hooks rather than pane scraping** — [agent-of-empires](https://github.com/agent-of-empires/agent-of-empires) (`src/hooks/status_file.rs`).
  Claude hooks write `running`/`waiting`/`idle` to per-instance sidecar files, and pane reconciliation fills the gaps.
  For LoomYard: shuttle already reads the Stop payload; adding a "waiting for input" sidecar could replace screen heuristics.
- **Reference status patterns and a paste pitfall** — CAO (`providers/claude_code.py`, `docs/inbox-delivery.md`).
  Regexes cover the spinner, the idle prompt, `↑/↓ to navigate` (an open question), plan approval, the trust and bypass dialogs, and "Newer model available".
  Bracketed paste behaves differently in tmux below and above 3.7.
  For LoomYard: compare against shuttle's startup-gate and readiness handling and creel inject.
- **Context forecasting and rotation for every session** — ntm (`docs/context-forecast.md`, `docs/compaction-recovery.md`).
  It warns at over 70% occupancy with under 15 minutes projected, and re-injects a refresher after "Conversation compacted".
  For LoomYard: the orch watcher's cycle, extended to the long-lived webster Master and the loom driver.
- **Edit-loop detector** — Bernstein (`docs/operations/agent-loop-detection.md`).
  It kills an agent that edits the same file 4 or more times within 5 minutes.
  For LoomYard: reed's watchdog or shuttle.
- **Destructive-command policy as a PreToolUse hook** — ntm (README §5).
  It is injected via `--settings` on every launch and holds even in bypass mode.
  For LoomYard: shuttle launch settings, if the agents' deny lists do not already cover this.
- **Staggered spawns** — ntm spaces launches by learned rate limits (README only, algorithm not read).
  For LoomYard: webster fork pacing and batten fan-out.

## Observability and audit (report, weft)

- **Hash-chained, replay-verifiable run journal** — Bernstein (`docs/operations/deterministic-replay.md`).
  `bernstein replay latest --verify` names the first step that diverges, and an Ed25519 run receipt verifies offline.
  For LoomYard: a hash chain over producer outputs in weft would make an archived `archive/<slug>/<tip>` tag self-verifying.
- **Unverified-completion rate** — Bernstein (`docs/operations/verification-tracking.md`).
  It alerts when more than 30% of completions in a window had no test, gate or signal check.
  For LoomYard: a new `lyx report` view.
- **Commit-to-record links via git notes** — [AgentNote](https://github.com/wasabeef/AgentNote).
  It attaches the session record behind each commit as a git note on a dedicated ref, plus a commit trailer.
  For LoomYard: a trailer or note on each warp batch commit pointing at its weft record path, so `git log` on the code leads to the run.
- **Run diff** — ccswarm `run diff <a> <b>`.
  For LoomYard: `lyx report` comparing two archived runs' timelines and token use.
- **OpenTelemetry tracing** — Conductor.
  For LoomYard: optional spans per producer, alongside `report timing`.

## UI and notifications (orch, tender, board)

- **Web dashboard and push notifications** — agent-of-empires (`docs/guides/web-dashboard.md`, `docs/push-notifications.md`) and ralphex `--serve` (SSE live output, late-join history).
  agent-of-empires adds tunnel access with a rotating token, a phone QR pairing code, a read-only mode, and Web Push on waiting/error with a cooldown.
  For LoomYard: a read-only view over board, `shed status` and creel, and a push when a run reaches PR-Gate, circling or a halt.
  [ralphex notifications](https://github.com/umputun/ralphex) (Slack, Telegram, webhook, custom script) are a lighter first step.
- **Attention feed with acknowledgement states** — ntm (`docs/attention-feed-contract.md`).
  It is a durable stream of items needing the human, each NEW/SEEN/SNOOZED/ACK/DISMISSED, with cursors.
  For LoomYard: a single "what is waiting on me" list across runs for the operator and the orch.
- **Terminal bell on a waiting gate** — Conductor's fleet TUI.
  For LoomYard: a near-free signal from reed when PR-Gate or a parent review is waiting.

## Spec and discussion review (Discussion-Write, Discussion review, stencils)

- **Requirements-quality checklist** — Spec Kit `/checklist` ("unit tests for English").
  Items tagged Completeness/Clarity/Consistency/Measurability/Gap/Traceability carry IDs, cite requirements, and are answered by the reviewer, never ticked by the writer.
  For LoomYard: an artifact the Discussion-review judge must answer item by item.
- **Fixed analysis categories** — [Kiro Analyze Requirements](https://kiro.dev/docs/specs/analyze-requirements/).
  It checks for inconsistency, ambiguity, functional/non-functional conflicts, unstated assumptions and missing edge cases.
  For LoomYard: name these lenses in the Discussion-review stencil.
- **Versioned constitution fed to reviewers** — Spec Kit `constitution`.
  It carries a semver version, amendment dates and a sync-impact report, and its MUST rules feed every analysis.
  For LoomYard: PATTERN.md is enforced by Go tests; handing its rules to plan and code reviewers as numbered MUSTs is the new part.
- **Glob-scoped steering context** — [Kiro steering](https://kiro.dev/docs/steering/).
  Context files load always, on a `fileMatch` glob, manually, or by description match, and support `#[[file:path:12-28]]` includes.
  For LoomYard: stencils could pull in per-area guidance chosen by the paths of a plan's glyphs.
- **Living spec and critique before execution** — [Augment Intent](https://www.augmentcode.com/intent).
  A Critique specialist reviews the spec for feasibility before implementers start, and the spec updates as work lands.
  LoomYard's Discussion and Plan reviews already cover the critique.
  Intent is closed source and Augment says Critique's checks are "not yet fully documented", so nothing more is verifiable.

## Projects surveyed

| Project | Link | License | Last push | One line |
|---|---|---|---|---|
| Fusion | https://github.com/Runfusion/Fusion | MIT | 2026-10-15 | Multi-node TS "software factory": kanban, missions, plan/review/execute gates, symbol leases. |
| ORCH | https://github.com/oxgeneral/ORCH | MIT | 2026-08-01 | CLI/TUI for agent "companies" with a task transition table and goals. |
| Bernstein | https://github.com/sipyourdrink-ltd/bernstein | Apache-2.0 | 2026-10-16 | Python deterministic scheduler with a governance, audit-journal and replay layer. |
| Conductor | https://github.com/microsoft/conductor | MIT | 2026-10-15 | YAML multi-agent workflows with deterministic routing, limits and a dashboard. |
| agent-of-empires | https://github.com/agent-of-empires/agent-of-empires | MIT | 2026-10-15 | Rust tmux session manager with web/phone dashboard, hook-based status and sandboxes. |
| dmux | https://github.com/standardagents/dmux | MIT | 2026-08-16 | tmux pane per worktree with lifecycle hooks and merge handlers. |
| ntm | https://github.com/Dicklesworthstone/ntm | NOASSERTION (custom) | 2026-10-15 | Go tmux swarm manager: robot mode, reservations, context forecasting, attention feed. |
| CLI Agent Orchestrator (CAO) | https://github.com/awslabs/cli-agent-orchestrator | Apache-2.0 | 2026-10-16 | Supervisor/worker over tmux with MCP handoff/assign tools and cron flows. |
| oh-my-claudecode | https://github.com/Yeachan-Heo/oh-my-claudecode | MIT | 2026-10-16 | Claude Code plugin with team pipeline, ralph loop, verify baseline, rate-limit wait. |
| ccswarm | https://github.com/nwiizo/ccswarm | MIT | 2026-10-13 | Rust YAML workflow engine with a quorum review stage and harness scenarios. |
| claude-squad | https://github.com/smtg-ai/claude-squad | AGPL-3.0 | 2026-08-20 | Go TUI for worktree+tmux agent sessions with pause/resume and auto-yes. |
| ralphex | https://github.com/umputun/ralphex | MIT | 2026-10-13 | Go plan executor with fresh session per task and multi-phase cross-model review. |
| GitHub Spec Kit | https://github.com/github/spec-kit | MIT | 2026-10-15 | Spec-driven development toolkit: constitution, clarify, analyze, checklist, converge. |
| Traycer | https://github.com/traycerai/traycer | MIT | 2026-10-16 | Plan/phase workspace with severity-tagged verification and YOLO epics. |
| Kiro | https://github.com/kirodotdev/Kiro | none (issue tracker; product closed) | 2026-09-15 | AWS spec-driven agentic IDE: EARS requirements, property-based tests, steering. |
| Augment Intent | https://www.augmentcode.com/intent | proprietary | n/a | Desktop multi-agent workspace with a Critique spec-review specialist. |
| foremerge | https://github.com/naw103/foremerge | Apache-2.0 | 2026-10-14 | Rust intent-declaration protocol with deterministic semantic conflict rules. |
| yylo | https://github.com/yylo-dev/yylo | MIT | 2026-10-15 | Orchestration CLI with typed boundaries, risk tiers and a Git-native ledger. |
| tree-sitter-analyzer | https://github.com/aimasteracc/tree-sitter-analyzer | MIT | 2026-10-16 | Tree-sitter impact and safe-to-edit analysis. |
| AgentNote | https://github.com/wasabeef/AgentNote | MIT | 2026-07-13 | Agent session records attached to commits as git notes. |
| GNAP | https://github.com/farol-team/gnap | MIT | 2026-03-17 | Git-native agent protocol (RFC draft); fabric/weft already covers it. |

Not verified:
- the internals of Intent's Critique, and Kiro's analysis method (both closed source);
- Fusion's routines and oversight levels (README only);
- ntm's smart stagger algorithm;
- whether ccswarm's container mode is wired in;
- Bernstein's policy-as-code enforcement path;
- whether Traycer's repo holds product code.

Fresh searches found no open-source project that validates whole agent plans at symbol level before execution, so quarry's glyph validation looks ahead of what is public.

## Sources

- https://github.com/andyrewlee/awesome-agent-orchestrators
- https://github.com/Runfusion/Fusion — `docs/missions.md`, `docs/workflow-steps.md`, `docs/settings-reference.md`, `packages/core/src/tasks/symbol-lock-types.ts`, `packages/core/src/tasks/file-scope-lease.ts`, `packages/core/src/planner/plan-approval.ts`
- https://github.com/oxgeneral/ORCH — `src/domain/transitions.ts`, `src/domain/scope.ts`, `src/domain/goal.ts`, `src/application/orchestrator.ts`
- https://github.com/sipyourdrink-ltd/bernstein — `docs/operations/{deterministic-replay,cost-envelopes,checkpointed-retries,fork-from-step,escalation-ladder,agent-loop-detection,abandonments,verification-tracking,stall-escalation}.md`, `src/bernstein/core/quality/janitor.py`, `src/bernstein/core/security/quarantine.py`
- https://bernstein.run/
- https://github.com/microsoft/conductor — `docs/workflow-syntax.md`, `docs/cli-reference.md`, `docs/configuration.md`
- https://opensource.microsoft.com/blog/2026/05/14/conductor-deterministic-orchestration-for-multi-agent-ai-workflows/
- https://github.com/agent-of-empires/agent-of-empires — `src/hooks/status_file.rs`, `docs/guides/web-dashboard.md`, `docs/push-notifications.md`, `docs/guides/sandbox.md`
- https://github.com/standardagents/dmux — `src/utils/hooks.ts`, `context/HOOKS.md`, `src/utils/claudeHooks.ts`, `src/actions/merge/conflictResolution.ts`
- https://github.com/Dicklesworthstone/ntm — `docs/context-forecast.md`, `docs/compaction-recovery.md`, `docs/attention-feed-contract.md`, `docs/worktree-merge-preview.md`, `internal/health/health.go`
- https://github.com/awslabs/cli-agent-orchestrator — `src/cli_agent_orchestrator/mcp_server/server.py`, `src/cli_agent_orchestrator/providers/claude_code.py`, `docs/inbox-delivery.md`, `docs/flows.md`
- https://github.com/Yeachan-Heo/oh-my-claudecode — `skills/ralph/SKILL.md`, `src/features/rate-limit-wait/daemon.ts`, `src/features/notepad-wisdom/types.ts`
- https://github.com/nwiizo/ccswarm — `crates/ccswarm/src/workflow/sangha.rs`, `crates/ccswarm/src/workflow/judge.rs`, `docs/HARNESS_ENGINEERING.md`
- https://github.com/smtg-ai/claude-squad — `session/instance.go`, `daemon/daemon.go`
- https://github.com/umputun/ralphex — README, `pkg/config/defaults/prompts/codex.txt`, `pkg/config/defaults/agents/`, `docs/notifications.md`
- https://github.com/github/spec-kit — `templates/commands/{analyze,checklist,clarify,converge,constitution}.md`
- https://docs.traycer.ai/extension/tasks/verification.md
- https://docs.traycer.ai/tasks/yolo-mode.md
- https://docs.traycer.ai/concepts/artifact-workflow.md
- https://kiro.dev/docs/specs/
- https://kiro.dev/docs/specs/correctness/
- https://kiro.dev/docs/specs/bugfix-specs/
- https://kiro.dev/docs/specs/analyze-requirements/
- https://kiro.dev/docs/steering/
- https://www.augmentcode.com/intent
- https://www.augmentcode.com/tools/best-ai-spec-review-tools-for-development-teams
- https://github.com/naw103/foremerge — README, `docs/conflict-detection.md`, `src/conflict.rs`
- https://github.com/yylo-dev/yylo — `src/templates/config/risk-policy.json`, `src/templates/scripts/risk_policy.py`, `docs/lifecycle-simplification-migration.md`
- https://github.com/aimasteracc/tree-sitter-analyzer
- https://github.com/wasabeef/AgentNote
- https://github.com/farol-team/gnap
