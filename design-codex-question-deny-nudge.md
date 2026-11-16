# A Codex session gets the question-tool deny and the batching nudge Claude runs

`codex-question-deny-nudge` · Task · feature, codex, shuttle, stencil, heddle · draft

Decision record: [Sketch](decision-codex-question-deny-nudge.md)

Depends on: [`codex-standing-instructions`](design-codex-standing-instructions.md)

Filed from the Sketch `codex-same-as-claude`.

Takes: codex-standing-instructions' codexengine launch line, whose one hooks.PreToolUse and hooks.PostToolUse settings these entries join
Delivers: the request_user_input deny on an autonomous Codex launch; the Codex batching nudge reading the rollout's top-level calls and exec cells, judged by Claude's rules and keyed by transcript_path; the read-only lookup check moved out of claudeengine into a package both engines import; codex.yaml's batching_nudge block
Exports: the moved lookup check's package under internal/shuttleengine

The part changes these packages:

- internal/shuttleengine/codexengine (about 800 changed lines)
- internal/shuttleengine/claudeengine (about 80 changed lines)
- internal/shuttleengine/bashlookup (about 280 changed lines)
- internal/shuttlecli (about 60 changed lines)
- internal/shuttleengines (about 40 changed lines)
