# heddle file targets: split claude-file and codex-file, add a shared target for both

`heddle-split-file-targets` · Note · enhancement, heddle

Imported from [issue #104](https://github.com/Knatte18/loomyard/issues/104).

A heddle entry with target claude-file is rendered into both CLAUDE.local.md (Claude Code) and AGENTS.override.md (Codex). The name says Claude only, and there is no way to give an instruction to one agent family alone, although some instructions apply to Codex and not to Claude, or the other way round.

Proposal: three file targets.

- claude-file renders into CLAUDE.local.md only.
- codex-file renders into AGENTS.override.md only.
- A new shared target (for example agents-file) renders into both.

Existing claude-file entries keep today's behaviour by migrating to the shared target. lym heddle set --help, lym heddle list and HEDDLE.md state which files each file target renders to.