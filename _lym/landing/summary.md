# Map every CLAUDE.md rule to where lym's agents already get it

Adds `docs/research/claude-md-rule-map-2026-10-29.md`, a per-rule table the operator decides from before `CLAUDE.md` is deleted.
The commit adds that one file and changes no other.

The report opens with how `CLAUDE.md` reaches agents: Claude Code loads it as an instruction file in every session in the repository, and three stencils tell a role to read it.
It then lists which roles receive `PATTERN.md` by injection, each with the `pattern.Directive` call site, and which roles have no `pattern_directive` marker.

The table has one row per rule or sentence of `CLAUDE.md`, with the rule quoted, where it already stands as `file:line` or `none`, the agents that need it, and a verdict of `covered`, `move to <place>` or `drop`, each with a reason.
Two further sections name the rules whose only home is `CLAUDE.md` and the rules a check enforces in code.

## Design decisions

- Verdicts are recommendations; a rule for every use of lym gets `move to` a stencil, a PATTERN entry or the orch rules file, following the placement rule in `CLAUDE.md` itself.
- A rule a hand push or an operator session needs, which no lym role performs, goes to the orch rules file rather than a stencil.
- Cited line numbers are a snapshot at this commit.

## Findings the follow-up must act on

- `contracts/stencils/darn/darn-template-write.md`, `contracts/stencils/drift/drift-template-baseline.md` and `contracts/stencils/orch/orch-template-role.md` name `CLAUDE.md` and need an edit when it goes.
- `internal/websterengine/merriambase.go` counts `CLAUDE.md`'s size in Merriam's start-base calibration.
- Two rules have no check today: the interactive-tmux rule and the filesystem-links rule.
