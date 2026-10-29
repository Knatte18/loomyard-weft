# Delete the loomyard CLAUDE.md after moving any rule only it holds

`claude-md-lym-rules-out` · Task · enhancement, stencil, pattern, orch · running · Finalize

## Goal

Delete the loomyard `CLAUDE.md` after each rule that stands only in it has a new home, as the operator decided from `docs/research/claude-md-rule-map-2026-10-29.md` (2026-10-29).
LoomYard adds no instruction file to a repository (`warp-visibility`), and the rules agents follow reach them through PATTERN, the stencils and the plugin skills.

## Scope

Move these rules, each to the place the report names:

- "A new cross-cutting invariant goes into PATTERN in the same commit" and the docs-in-the-same-commit list: into `contracts/stencils/darn/darn-template-write.md` step 4.3 and `contracts/stencils/drift/drift-template-baseline.md`, whose lines 26, 76 and 22 name `CLAUDE.md` as the only source of the list; the pointers go.
- "Never run `update-plugins.sh` unasked, never install `lym` elsewhere": into the hub-wide orch rules file through `lym orch rules`, starting from a copy of the current `_board/_lym/orch/rules.md`.
- The placement rule for where a new rule goes: rewrite the pointer at `contracts/stencils/orch/orch-template-role.md:186` to name the orch rules file and the stencils.
- The `--upto 2` gate before a push to `main`, the own-Bash-call rule and the docs-only exemption: into the same orch rules file, for a session that pushes by hand.
- "Work only in your own worktree: never edit, commit or push in another, and never create one unless the user says so": into `contracts/stencils/edit/edit-directive.md`, which every spawned role receives.

Drop the opening line about LoomYard developing itself and the line on the conversational shorthands.
Leave the rows the report marks `covered` as they stand.

Then delete `CLAUDE.md` and edit the comment at `internal/websterengine/merriambase.go:22`, whose `merriamFixedContext` counts `CLAUDE.md`'s size in Merriam's start base, so the constant matches the base without the file.
The interactive-tmux rule and the filesystem-links rule move through the task `pattern-spawn-and-links-scans`, which lands first.
