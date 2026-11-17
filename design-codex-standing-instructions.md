# A Codex session gets Claude's standing items, user setup, instruction files and subagent terms in its developer instructions

`codex-standing-instructions` · Task · feature, codex, shuttle, stencil, heddle · running · Plan-Burler-Review

Decision record: [Sketch](decision-codex-standing-instructions.md)

Filed from the Sketch `codex-same-as-claude`.

Takes: prompt-engine-per-launch's prompt engine on the spec, so Prepare knows the launch renders for Codex
Delivers: Codex developer_instructions in D5's order with the 128 KiB refusal; the agents table's subagent effort and thread cap; the Claude user setup (global file, CLAUDE.md files above the root, skill listing, plugin session-start contexts) resolved in claudeengine and handed to codexengine through a shuttleengine seam; the leaf @-import expander that claudeengine and heddle share; project_doc_max_bytes and AGENTS.override.md carrying both files; the agents-prompt target, the two shipped entries on agents-file and the hub reconcile rendering file targets
Exports: the plugin session-start hook runner and Codex's sandbox probe as injectable seams of Prepare, so launch-manifest-parity can pass a faked runner and probe and a runner that lists each hook unrun; Codex text it adds in Go literals names none of D2's closed terms

The part changes these packages:

- internal/shuttleengine/codexengine (about 600 changed lines)
- internal/shuttleengine/claudeengine (about 400 changed lines)
- internal/shuttleengine (about 120 changed lines)
- internal/shuttleengines (about 60 changed lines)
- internal/claudeimports (about 150 changed lines)
- internal/heddleengine (about 350 changed lines)
- internal/heddlecli (about 30 changed lines)
- internal/hubreconcile (about 80 changed lines)
