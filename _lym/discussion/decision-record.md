# Decision record — claude-md-rule-map

## Goal

Land one research report, `docs/research/claude-md-rule-map-2026-10-29.md`, that maps every rule of the repository's `CLAUDE.md` to where lym's agents already receive it, so the operator can decide per rule what happens to it before `CLAUDE.md` is deleted.

## Scope

- Input is `CLAUDE.md` alone, read by this one writer session; no other runs, models or repeats.
- The writer reads the sources itself: `CLAUDE.md`, `PATTERN.md`, `pattern/`, `contracts/stencils/`, `plugins/`, the orch role stencil and the code that injects directives.
- The report opens with which roles receive `PATTERN.md` by injection and which do not, citing the injecting code.
- Then one table row per rule or sentence of `CLAUDE.md`, with columns: the rule in a short quote, where it already stands for the agents that need it (`file:line` or `none`), the agents that need it, and a verdict of `covered`, `move to <place>` or `drop`, each with a one-line reason.
- The report names every rule whose only home is `CLAUDE.md` and which agents lose it on deletion, and every rule enforced in code with the check that enforces it.
- Changes no other file: `CLAUDE.md`, `PATTERN.md`, stencils and docs stay as they are.

## Decisions

### D1 — One new file, and nothing else changes

The commit adds `docs/research/claude-md-rule-map-2026-10-29.md` and touches no other tracked file.
Rationale: the board entry scopes the report as the sole change, and the follow-up task acts on it.

### D2 — Injection section comes first and is derived from the code

The report's first section lists, per role, whether the PATTERN overview is injected, and cites the call site and the stencil it fills.
Rationale: a rule's coverage by `PATTERN.md` only counts for the roles that receive it, so every row's "agents that need it" is judged against this list.

### D3 — A row per sentence-level rule, in `CLAUDE.md` order

Each rule or sentence of `CLAUDE.md` gets its own row, in file order, so the operator can decide each one.
A rule's coverage cites `file:line` read in this worktree, or `none`.
Rationale: the operator's choice per row is the report's purpose, and file order lets them check it against the source.

### D4 — Verdicts follow the file's own placement rule

A rule for every use of lym, in any repository, gets `move to` the stencil, skill or code check that reaches the agents needing it.
A rule only for developing LoomYard, with no agent that would lose it, gets `drop` or `covered` by the place that already states it.
Rationale: `CLAUDE.md` states where each kind of rule belongs, and a verdict that follows it needs no judgment the operator did not already make.

### D5 — Enforced-in-code rules name the check

For each rule a test, scan or refusal enforces, the report names that check by file and symbol.
Rationale: a rule enforced in code loses nothing on deletion, and the operator must see which checks that is.

## Constraints

- PATTERN entries bind any code change; this change edits no code, so none is touched.
- Markdown follows semantic line breaks: one sentence per line, table cells on one line.
- A cited `file:line` is read from the worktree at the time of writing, never recalled.

## Auto-mode assumptions

- No operator is present; the report's verdicts are recommendations the operator decides.
- A rule stated in `CLAUDE.md` and also in `README.md` counts as stated in README, which agents do not receive by injection.

## Open risks

- A line number cited in the report goes stale when the cited file changes; the report is a snapshot of this commit.
- A rule's "agents that need it" is the writer's reading of the role stencils, not a measured delivery.

## Acceptance criteria

- D1: the diff holds exactly one added file, `docs/research/claude-md-rule-map-2026-10-29.md`.
- D2: the report's first section lists each PATTERN-injected role and each role without it, each citing code.
- D3: every rule and sentence of `CLAUDE.md` has a row carrying the quote, the coverage cite or `none`, the agents and a verdict with a reason.
- D4: every verdict is one of `covered`, `move to <place>` or `drop`, and each `move to` names a concrete place.
- D5: the report names every rule whose only home is `CLAUDE.md` and every rule a check enforces, with the check.
