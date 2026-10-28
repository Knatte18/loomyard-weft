# Codex CLI as a second shuttle engine: codexengine

`codex-engine` · Note · enhancement, shuttle, codex

Codex CLI is the candidate second engine for shuttle, to offload Claude token use: Loomyard spawns hundreds of agents a day (operator, 2026-10-13), and Codex is more relevant to support than Gemini (operator, 2026-10-03).
The hands-on spike is done: `docs/research/codex-spike-2026-10-24/report.md` holds every finding, a proposed launch line, a pane classifier table and a send sequence.
The stencil audit is done too: `docs/research/stencil-engine-audit-2026-10-24/report.md` proposes 13 include blocks over 127 stencil sites, 8 spec sites and 35 Go sites; blocks await-completion, message-agent and session-lifecycle have no tested Codex answer, and an engine-keyed choice of include set at render time is missing.
Its `Not tested` section stays open: billing under a ChatGPT login, a `PermissionRequest` `allow` decision and `~/.agents/skills`; `codex exec` and `Tab` queueing are covered by `docs/research/codex-hook-delivery-2026-10-28/report.md` (task `codex-hook-spike`): `codex exec` runs the same `post_tool_use` and `Stop` hooks, and a `Tab`-queued line reaches the model after the turn ends, not at the next step.

## Design basis (operator view, not a decision, 2026-10-24)

When claudeengine was built, what an engine needs was unknown; now claudeengine shows it, so `codexengine` is designed from claudeengine's list of needs (launch, startup classifier, send, idle and busy, completion, interrupt, usage, guards), each mapped to its Codex counterpart in the report.

## Permissions

- The launch line sets `-a never -s workspace-write`, as claudeengine sets its permission mode; the spike confirmed it stops every prompt. Saved approvals are never relied on: Codex's "allow for later" saves an exact command line, not a prefix (operator's own use, 2026-10-24).
- Operator view, not a decision: Codex's permission list mirrors Claude's, which allows broadly and forbids only destructive commands; a hand translation of `~/.claude/settings.json` sits in `~/.codex/rules/claude.rules` on Neve.
- An `allow` rule in a rules file runs its command outside the sandbox; a `forbidden` rule beats a broader allow.

## Folder trust and the Codex home (operator decisions, 2026-10-24)

Codex asks once per folder whether to trust it, keyed on the git repository root, and every lym worktree is a new root; a parent-folder entry does not cover a repository, and `-c` cannot set trust (tested on Codex 0.164).
- One shared lym Codex home per machine (such as `~/.lym/codex`) is every launch's `CODEX_HOME`: config, rules, hooks and skills rendered by lym, with links to `~/.codex/auth.json` and `~/.codex/sessions`; the operator's own `~/.codex` stays untouched. A `CODEX_HOME` with its own trust entry and the auth link started Codex with no prompt (tested 2026-10-24).
- `codexengine` adds the launch's worktree root to that home's trust list when missing, and in the same write drops entries whose folder no longer exists, so no teardown step is needed and only shuttle knows of it.
- Each write is a read-modify-write under an `internal/lock` file lock, written to a temp file and renamed into place.
- Codex writes `config.toml` itself too (a `[tui]` table on first start) and ignores the lock: lym pre-sets those settings, and checks after startup that the entry is still there.
- Tested 2026-10-28 (`codex-hook-spike`, one sample): one accepted trust entry keyed on the main checkout's root covers a linked worktree of the same repository, and the project hook ran there with no prompt; whether the shared lym Codex home still needs an entry per worktree is not tested.

## Steps

From `docs/research/codex-feasibility.md`, section 3, after the spike:

1. A minimal `codexengine` for the single-session roles: Codex home, launch line, hook-fed event parsing, startup classifier, send and paste handling, completion (`Stop` or `task_complete`) and interrupt (`turn_aborted`). Medium to large; claudeengine is about 3,900 non-test lines.
2. Skills sync into the Codex home, and the deny guardrails as Codex `PreToolUse` hooks (a hook that fails to run fails open). Medium.
3. Engine-keyed include sets: the stencil renderer picks the running engine's file of named blocks at render time, which only `seatengine` and `twilldirective` do today (`docs/research/stencil-engine-audit-2026-10-24/report.md`); Burler review's stencils need it (operator, 2026-10-24).
4. Forks and orch parity. Large; deferred. A twill-like fork fan is feasible, see the twill spike report.

## Plan (operator, 2026-10-27)

The Codex account is ChatGPT Pro (the 100 tier, plan `prolite` in the rollouts), bought for testing and benching, not yet for production use: every Codex bench states an exact, small cap on runs.
The weekly budget is getting used up: prefer a desk study over a new Codex run, and spend Codex runs sparingly (operator, 2026-10-27).
The forked Astra fan at medium effort with block v2 is the setup, and the next step is building, starting with step 1 below, whose design is discussed with the operator before it goes on the board (operator, 2026-10-27).
Pro has a weekly window and no 5-hour window; a forked Astra fan run on one plan costs a few points of the week.
Codex effort is at most `high`: `xhigh` and above cost too much (operator, 2026-10-27).
A Codex reviewer always gets the context a Claude reviewer of the same round gets, the PATTERN overview and the repository's agent instructions included (operator, 2026-10-27).

## Review bench (2026-10-25)

`docs/research/codex-review-bench-2026-10-25/report.md`: over 10 discussions and plans at medium effort, solo Codex does not reach the Claude lens fan's coverage.
`gpt-6.1-sol` found 22 of 134 Claude findings (7 of 21 blocking), matched 9 of 10 verdicts and added 11 real findings Claude missed; Luna is too shallow, Astra matched Sol at about 4 times the plan cost.
The report suggests Sol as an extra, non-gating reviewer beside the Claude fan, not a replacement; the plan's 5-hour limit stopped the bench early, so repeats are unmeasured; higher effort is measured on one plan by the Astra benches below.

`docs/research/codex-astra-solo-bench-2026-10-25/report.md` (landed as `codex-astra-solo-bench`) benches Astra against solo Opus, both at medium effort, on two plans: each reproduced 4 of the Claude fan's 23 findings, Opus 1 of its 5 blocking and Astra none.
Astra was 2.3 to 3 times faster and cost 22 points of the Plus plan's 5-hour window on one plan; it blocked both plans as the fan did, where Opus approved one the fan blocked.
Two inputs, one run per cell, one grader: it ranks neither reviewer.

Five follow-up benches of 2026-10-27, all on plan `P4` (`machine-handover`), one run per cell, one grader, against the Claude fan's 11 findings (3 `BLOCKING`); each returned the fan's verdict `BLOCKING` and no false finding, and none ranks Astra against the fan:
- `docs/research/codex-astra-fan-bench-2026-10-27/report.md`: a cold-call Astra lens fan at medium effort found 0 of 11, with 8 unique real findings; 466k fresh input tokens and 907 s against solo's 77k and 166 s.
- `docs/research/codex-astra-solo-high-bench-2026-10-27/report.md`: solo Astra at high effort found 1 of 11 where medium found 2; about 2.9 times the wall time, a different set of findings, not a larger one.
- `docs/research/codex-astra-fork-fan-bench-2026-10-27/report.md`: a handler with real forks (`spawn_agent`, `fork_turns="all"`) at high effort found 1 of 11; the forks ran concurrently, children of the handler's thread, as the twill spike found.
- `docs/research/codex-astra-context-parity-bench-2026-10-27/report.md`: the same forked fan with the PATTERN overview inlined and the snapshot's `CLAUDE.md` loaded as `AGENTS.md` found 2 of 11, none `BLOCKING`; the gain is the PATTERN-fit finding (fan `F6`).
- `docs/research/codex-astra-tuned-prompt-bench-2026-10-27/report.md`: the forked fan with that context and one block of reading directions (read every `pattern/` background file, read the test card against the test-speed rules) found 4 of 11 and 2 of 3 `BLOCKING`, at 631k fresh input tokens and 950 s; it changes three things at once, so the block's share is unproven.
The `BLOCKING` finding no Astra run reached on `P4` is the owner guard on `retryWeftPush` reading origin's Bolt outside the reconcile.

Three runs of 2026-10-27 kept the tuned setup (context parity, the reading block word for word, forked fan) and varied the plan and effort, adding `P1` (`gate-runs-once`; Claude fan 12 findings, 2 `BLOCKING`):

| Plan, effort | Claude fan findings found | `BLOCKING` found | Unique real | False | Fresh input | Wall time |
|---|---|---|---|---|---|---|
| `P4` high | 4 of 11 | 2 of 3 | 6 | 0 | 631k | 950 s |
| `P4` medium | 3 of 11 | 2 of 3 | 3 | 0 | 236k | 492 s |
| `P1` high | 4 of 12 | 1 of 2 | 6 | 0 | 412k | 1114 s |
| `P1` medium | 3 of 12 | 0 of 2 | 5 | 0 | 263k | 414 s |

Reports: `docs/research/codex-astra-tuned-p4-med-bench-2026-10-27/`, `codex-astra-tuned-p1-high-bench-2026-10-27/` and `codex-astra-tuned-p1-med-bench-2026-10-27/` under `docs/research/`.
High found more than medium on both plans at about twice the wall time.
On `P1` the block, written after the `P4` misses, still reached the `PATTERN.md` headroom-order finding at high effort; no run reached a finding that needs every reader of a changed symbol enumerated (`P1` fan `F2`) or a read of code beyond the cards' own (`P4` fan `F1`).
Across every Astra run no finding was false, and each found real defects the Claude fan lacks.

Three benches of 2026-10-27 tried block v2 (reading directions revised) with the fork cap raised from 6 to 20 tool calls, one input each, one grader, one run per cell:
- `docs/research/codex-v2-p1-bench-2026-10-27/report.md` (`P1`): high found 6 of 12 fan findings and 2 of 2 `BLOCKING`; two medium runs found 5 and 6, so effort and chance are not separable. Block v1 found 4 and 3. The cap of 20 never bound (most commands in one fork: 12). It is the first run to reach `F2`, the enumerated readers of a changed symbol, in part.
- `docs/research/codex-v2-p4-bench-2026-10-27/report.md` (`P4`): high found 3 of 11 and 1 of 3 `BLOCKING`, medium runs 2 and 3; block v2 did not beat v1 (4 and 3), and no run reached `F1`, the owner guard on `retryWeftPush`. The two medium runs share one fan finding and each holds a `BLOCKING` finding the other lacks.
- `docs/research/codex-p2-gap-texts-bench-2026-10-27/report.md` (`P2`, `quarry-csharp`, Claude fan 13 findings): adding three texts from `docs/research/claude-review-gaps-2026-10-27/report.md` (Proposals 1, 2 and 4) did not measurably change the yield (12 findings at 500k fresh tokens against 10 at 429k); the `way-forward` fork found nothing in any `P2` run, and one run returned `APPROVED` where the fan blocks.
None ranks a setting; two medium runs of one setting differ as much as any setting from another.

`docs/research/codex-code-review-bench-2026-10-28/report.md`, the first bench over code: the forked Astra fan at medium with block v2 reviewed two landed darn diffs twice each against the Claude Code-Review fan (`sonnet` at medium).
It found 2 of 5 and 3 of 6 fan findings, the same ones in both runs of an input, and missed the fan's one `BLOCKING` finding; it misses divergences from the decision record and acceptance criteria, and on one input it found two real PATTERN violations the fan lacks.
It does not stand in for the Claude Code-Review fan as it ran there.

## Goal (operator, 2026-10-27)

Codex is to take review load off Claude: the work is to tune Codex reviews until a Codex fan can stand in for a Claude fan, not to add Codex as an extra reviewer beside it.

## First candidate roles

Decision (operator, 2026-10-24): Codex's first role is Burler review only, so Claude plans and writes code while another model family reviews it.
The sole aim is to offload Claude.
Measured 2026-10-24 over the 10 latest full loom runs (`tools/tokencount/tokencount.sh -last 10`): the reviewers and their forks cost about 25% of all spend ($184 of $741), mostly Plan review (13%) and Discussion review (9%), while Code and Merge review are under 3%.
Over 12 non-docs darn runs of 2026-10-24 ($6.53 per run, orch excluded), the darn writer took 58% and the Code-Review reviewer with its forks 18%.
If a twill fan on Codex works poorly, the review runs as strong single Codex agents instead.
Codex reviews are investigated and benched thoroughly before any of it goes into production (operator, 2026-10-24).
Operator view, not a decision (2026-10-24): Codex's strong model Astra could also write the plan, likely single-shot rather than as a twill fan; the Plan writer and its forks are about 23% of spend, and a Codex-written plan would then be reviewed by Claude.

Single-session roles that spawn no forks and only write output files: Burler review chair without a cluster fan, Burler fixer, Describe, Bouncer judge and seed, treadle, friction, merge-conflict resolver, darn writer.
Webster, twill, the orch and the Burler cluster fan depend on cross-session messaging and background-task wake-ups, and wait.
Forks are no longer a blocker for a fan: `docs/research/codex-twill-spike-2026-10-24/report.md` found that Codex forks (`fork_turns: "all"`) run in parallel, three at once by default, and report back to the parent, with caveats (no per-fork model, no isolation, a fan is a request not a guarantee).

## Groundwork

The engine-neutral groundwork of the operator's 2026-10-15 decision (engine registry, config seams, model choice, usage readers, creel in three stages) has only creel stage 3, note `creel-agents`, left on the board; it matters only once Codex takes a role beyond Burler review, since such roles message through Claude Code's SendMessage today (operator, 2026-10-24).
The review-only first step waits on no creel work.

## Model aliases (operator, 2026-10-15)

`sonnet[high]` is engine `claude`, model `sonnet`, effort `high`, and Codex's models get the same form, such as `sol[medium]` for engine `codex`, model `sol`, effort `medium` (also Luna, Astra).
