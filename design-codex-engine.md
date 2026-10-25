# Codex CLI as a second shuttle engine: codexengine

`codex-engine` · Note · enhancement, shuttle, codex

Codex CLI is the candidate second engine for shuttle, to offload Claude token use: Loomyard spawns hundreds of agents a day (operator, 2026-10-13), and Codex is more relevant to support than Gemini (operator, 2026-10-03).
The hands-on spike is done: `docs/research/codex-spike-2026-10-24/report.md` holds every finding, a proposed launch line, a pane classifier table and a send sequence.
The stencil audit is done too: `docs/research/stencil-engine-audit-2026-10-24/report.md` proposes 13 include blocks over 127 stencil sites, 8 spec sites and 35 Go sites; blocks await-completion, message-agent and session-lifecycle have no tested Codex answer, and an engine-keyed choice of include set at render time is missing.
Its `Not tested` section stays open: billing under a ChatGPT login, `codex exec`, `Tab` queueing, a `PermissionRequest` `allow` decision and `~/.agents/skills`.

## Design basis (operator view, not a decision, 2026-10-24)

When claudeengine was built, what an engine needs was unknown; now claudeengine shows it, so `codexengine` is designed from claudeengine's list of needs (launch, startup classifier, send, idle and busy, completion, interrupt, usage, guards), each mapped to its Codex counterpart in the report.

## Permissions

- The launch line sets `-a never -s workspace-write`, as claudeengine sets its permission mode; the spike confirmed it stops every prompt. Saved approvals are never relied on: Codex's "allow for later" saves an exact command line, not a prefix (operator's own use, 2026-10-24).
- Operator view, not a decision: Codex's permission list mirrors Claude's, which allows broadly and forbids only destructive commands; a hand translation of `~/.claude/settings.json` sits in `~/.codex/rules/claude.rules` on Neve.
- An `allow` rule in a rules file runs its command outside the sandbox; a `forbidden` rule beats a broader allow.

## Folder trust and the Codex home (operator decisions, 2026-10-24)

Codex asks once per folder whether to trust it, keyed on the git repository root, and every lyx worktree is a new root; a parent-folder entry does not cover a repository, and `-c` cannot set trust (tested on Codex 0.164).
- One shared lyx Codex home per machine (such as `~/.lyx/codex`) is every launch's `CODEX_HOME`: config, rules, hooks and skills rendered by lyx, with links to `~/.codex/auth.json` and `~/.codex/sessions`; the operator's own `~/.codex` stays untouched. A `CODEX_HOME` with its own trust entry and the auth link started Codex with no prompt (tested 2026-10-24).
- `codexengine` adds the launch's worktree root to that home's trust list when missing, and in the same write drops entries whose folder no longer exists, so no teardown step is needed and only shuttle knows of it.
- Each write is a read-modify-write under an `internal/lock` file lock, written to a temp file and renamed into place.
- Codex writes `config.toml` itself too (a `[tui]` table on first start) and ignores the lock: lyx pre-sets those settings, and checks after startup that the entry is still there.
- Untested: whether Codex resolves a linked worktree to its own root or to the main repository's.

## Steps

From `docs/research/codex-feasibility.md`, section 3, after the spike:

1. A minimal `codexengine` for the single-session roles: Codex home, launch line, hook-fed event parsing, startup classifier, send and paste handling, completion (`Stop` or `task_complete`) and interrupt (`turn_aborted`). Medium to large; claudeengine is about 3,900 non-test lines.
2. Skills sync into the Codex home, and the deny guardrails as Codex `PreToolUse` hooks (a hook that fails to run fails open). Medium.
3. Engine-keyed include sets: the stencil renderer picks the running engine's file of named blocks at render time, which only `seatengine` and `twilldirective` do today (`docs/research/stencil-engine-audit-2026-10-24/report.md`); Burler review's stencils need it (operator, 2026-10-24).
4. Forks and orch parity. Large; deferred. A twill-like fork fan is feasible, see the twill spike report.

## Review bench (2026-10-25)

`docs/research/codex-review-bench-2026-10-25/report.md`: over 10 discussions and plans at medium effort, solo Codex does not reach the Claude lens fan's coverage.
`gpt-6.1-sol` found 22 of 134 Claude findings (7 of 21 blocking), matched 9 of 10 verdicts and added 11 real findings Claude missed; Luna is too shallow, Astra matched Sol at about 4 times the plan cost.
The report suggests Sol as an extra, non-gating reviewer beside the Claude fan, not a replacement; the plan's 5-hour limit stopped the bench early, so higher effort, repeats and Astra on plans are unmeasured.

## First candidate roles

Decision (operator, 2026-10-24): Codex's first role is Burler review only, so Claude plans and writes code while another model family reviews it.
The sole aim is to offload Claude.
Measured 2026-10-24 over the 10 latest full loom runs (`tools/tokencount/tokencount.sh -last 10`): the reviewers and their forks cost about 25% of all spend ($184 of $741), mostly Plan review (13%) and Discussion review (9%), while Code and Merge review are under 3%.
Over 12 non-docs darn runs of 2026-10-24 ($6.53 per run, orch excluded), the darn writer took 58% and the Code-Review reviewer with its forks 18%.
If a twill fan on Codex works poorly, the review runs as strong single Codex agents instead.
Codex reviews are investigated and benched thoroughly before any of it goes into production (operator, 2026-10-24).
Operator view, not a decision (2026-10-24): Codex's strong model Astra could also write the plan, likely single-shot rather than as a twill fan; the Plan writer and its forks are about 23% of spend, and a Codex-written plan would then be reviewed by Claude.

Single-session roles that spawn no forks and only write output files: Burler review chair without a cluster fan, Burler fixer, Describe, Bouncer judge and seed, treadle, friction, merge-conflict resolver, darn writer, the alignment agent.
Webster, twill, the orch and the Burler cluster fan depend on cross-session messaging and background-task wake-ups, and wait.
Forks are no longer a blocker for a fan: `docs/research/codex-twill-spike-2026-10-24/report.md` found that Codex forks (`fork_turns: "all"`) run in parallel, three at once by default, and report back to the parent, with caveats (no per-fork model, no isolation, a fan is a request not a guarantee).

## Groundwork

The engine-neutral groundwork of the operator's 2026-10-15 decision (engine registry, config seams, model choice, usage readers, creel in three stages) has only creel stage 3, note `creel-agents`, left on the board; it matters only once Codex takes a role beyond Burler review, since such roles message through Claude Code's SendMessage today (operator, 2026-10-24).
The review-only first step waits on no creel work.

## Model aliases (operator, 2026-10-15)

`sonnet[high]` is engine `claude`, model `sonnet`, effort `high`, and Codex's models get the same form, such as `sol[medium]` for engine `codex`, model `sol`, effort `medium` (also Luna, Astra).
