# Each launch writes its manifest, a parity test compares the engines per seat, and lym shuttle manifest prints both

`launch-manifest-parity` · Task · feature, codex, shuttle, stencil, heddle · running · Parent-Align

Decision record: [Sketch](decision-launch-manifest-parity.md)

Filed from the Sketch `codex-same-as-claude`.

Takes: prompt-engine-per-launch's per-producer render functions taking the engine; codex-standing-instructions' injectable hook runner and sandbox probe; every Codex delivery of codex-standing-instructions and codex-question-deny-nudge, which the parity test then finds present; codex-wording-engine-blocks' engine blocks, which the test renders per engine
Delivers: the launch manifest read back from each engine's Prepare, written as a record into the run directory and as a copy into the run-directory root's manifests directory, skipped by the orphan sweep and the run lookup by strand; the parity test with its producer table and the scan for unlisted launch-spec literals; the lym shuttle manifest verb; the bench rule on codex-engine-trial, and on codex-writer-bench, the draft that replaced codex-writer-bench-osl on 2026-11-16 and already runs the real producer, so D11's osl section lands there as a check that its body states the rule whole
Exports: the manifest record and the lym shuttle manifest verb, which the Codex benches read

The part changes these packages:

- internal/shuttleengine (about 300 changed lines)
- internal/shuttleengine/claudeengine (about 200 changed lines)
- internal/shuttleengine/codexengine (about 200 changed lines)
- internal/shuttlecli (about 200 changed lines)
- internal/shuttleparity (about 600 changed lines)
