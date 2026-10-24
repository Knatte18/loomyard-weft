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
3. Forks and orch parity. Large; deferred.

## First candidate roles

Single-session roles that spawn no forks and only write output files: Burler review chair without a cluster fan, Burler fixer, Describe, Bouncer judge and seed, treadle, friction, merge-conflict resolver, darn writer, the alignment agent.
Webster, twill, the orch and the Burler cluster fan depend on Claude Code's Agent forks, cross-session messaging and background-task wake-ups, and wait.

## Groundwork

The engine-neutral groundwork of the operator's 2026-10-15 decision (engine registry, config seams, model choice, usage readers, creel in three stages) has only creel stage 3, note `creel-agents`, left on the board; it matters since agents must message through creel rather than Claude Code's SendMessage before Codex can take a role.
The `codexengine` (guard table as Codex hooks, `codex.yaml`, the registry entry, the engine name, the aliases), the skills sync and tuning wait on `creel-agents`.

## Model aliases (operator, 2026-10-15)

`sonnet[high]` is engine `claude`, model `sonnet`, effort `high`, and Codex's models get the same form, such as `sol[medium]` for engine `codex`, model `sol`, effort `medium` (also Luna, Astra).
