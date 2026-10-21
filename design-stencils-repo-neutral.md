# Shipped review rubrics name loomyard's own PATTERN files, so reviewers in other hubs search for files that do not exist

`stencils-repo-neutral` · Task · bug, stencil, loom, burler

Imported from [issue #795](https://github.com/Knatte18/loomyard/issues/795).

Seen in the quarry hub (Knatte18/quarry, run csharp-alphabet), 2026-10-21.

The Plan-Burler-Review reviewer approved the plan, then started two background `find /` commands looking for `PATTERN-cli-cobra.md` and `PATTERN-test-economy.md`. Quarry has no PATTERN files.

Cause: the shipped stencils hard-code LoomYard's own PATTERN background files and `lyx` CLI rules, and every hub gets them. In `contracts/stencils/loom/loom-rubric-plan-review.md`: 'A card that adds or reshapes a `lyx` CLI command is checked against the audience bullet of `pattern/PATTERN-cli-cobra.md`' and '`PATTERN-test-economy` holds the rule'. The quarry hub's board copies of loom-rubric-plan-review, loom-rubric-webster-review, loom-rubric-merge-review, loom-rubric-darn-review, loom-template-plan, burler-step-fix and the loom-plan-spec carry such references.

Expected: shipped stencils name no repo-specific PATTERN file or lyx command. A rule that only holds for LoomYard comes from the hub's own config (lenses, PATTERN directive) or is dropped in hubs without a PATTERN.md.

## Issue comment, 2026-10-21

Operator's direction for the fix (2026-10-21): the PATTERN mechanism already leaves the choice to the plan agent, which reads the hub's own `PATTERN.md` and decides which background files touch the change. A shipped rubric or gate is general and cannot make that choice, so it names no PATTERN file, rule or `lyx` command at all. It checks the plan against the hub's `PATTERN.md` and the background files the plan itself names as relevant, and in a repo with no `PATTERN.md` that check does not apply. The LoomYard-specific checks (`PATTERN-cli-cobra`, `PATTERN-test-economy`) live in LoomYard's PATTERN files, where the plan agent finds them.