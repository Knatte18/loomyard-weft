# Claude-only wording moves into Claude engine blocks, each with a Codex copy, held by a term scan

`codex-wording-engine-blocks` · Task · enhancement, codex, shuttle, stencil, heddle · running · PR-Review

Decision record: [Sketch](decision-codex-wording-engine-blocks.md)

Filed from the Sketch `codex-same-as-claude`.

Takes: prompt-engine-per-launch's engine-aware renders, so an engine block resolves for the launch's engine; codex-standing-instructions' final heddle and codexengine text, which the scan then judges as landed
Delivers: every stencil and spec passage naming a closed-list term in a Claude engine block with a Codex copy; mode_rules and plan-gate finding text chosen per engine or worded for any; the D2 scan over stencils, specs and production Go literals, with orch places skipped, an allowlist with reasons for literals that never reach a Codex session, and a check that every engine block has a copy for every registered engine
Exports: nothing another part calls

The part changes these packages:

- contracts/stencils (about 1400 changed lines)
- contracts/specs (about 150 changed lines)
- internal/loomengine (about 120 changed lines)
- internal/websterengine (about 80 changed lines)
- internal/driftengine (about 40 changed lines)
- internal/burlerengine (about 40 changed lines)
- internal/darnengine (about 20 changed lines)
- internal/planparser (about 40 changed lines)
- internal/batcher (about 20 changed lines)
- internal/shellguard (about 40 changed lines)
- internal/seatengine (about 20 changed lines)
- internal/loomcli (about 20 changed lines)
