# Every producer renders its prompt and edit directive for the engine its launch resolves to

`prompt-engine-per-launch` · Task · enhancement, codex, shuttle, stencil, heddle · running · Drift-Verify

Decision record: [Sketch](decision-prompt-engine-per-launch.md)

Filed from the Sketch `codex-same-as-claude`.

Takes: nothing from another part; it waits for the board task weaver-implementer, which also changes internal/websterengine, so the two never run side by side
Delivers: a launch spec that carries its prompt engine, or the hand-written mark from lym shuttle run alone; a shuttleengine call that returns the engine name a model choice resolves to; Prepare refusing a mismatched or unset prompt engine; the orch spec pinned to Claude; one full edit directive per engine block with the shared stencil removed; the D4 scan over production launch-spec literals
Exports: for every producer outside seatengine, one prompt render function that takes the engine name and needs no session, split out of its launching function where that function fills the prompt inline (bouncer.go judgeCall and runSeedSpawn, websterengine, treadleengine, samplerengine, landingshed, frictionengine, mergeresolve); launch-manifest-parity's producer table calls these, and editdirective.Directive takes the engine

The part changes these packages:

- internal/shuttleengine (about 120 changed lines)
- internal/shuttleengine/claudeengine (about 40 changed lines)
- internal/shuttleengine/codexengine (about 40 changed lines)
- internal/shuttleengines (about 150 changed lines)
- internal/shuttlecli (about 40 changed lines)
- internal/orchcli (about 20 changed lines)
- internal/editdirective (about 40 changed lines)
- contracts/stencils (about 250 changed lines)
- internal/seatengine (about 40 changed lines)
- internal/sheddriver (about 30 changed lines)
- internal/treadleengine (about 60 changed lines)
- internal/samplerengine (about 40 changed lines)
- internal/websterengine (about 120 changed lines)
- internal/landingshed (about 40 changed lines)
- internal/gatefixer (about 30 changed lines)
- internal/loomcli (about 30 changed lines)
- internal/dobbyengine (about 30 changed lines)
- internal/shedarbiter (about 30 changed lines)
- internal/frictionengine (about 40 changed lines)
- internal/mergeresolve (about 40 changed lines)
- internal/shedadapters (about 120 changed lines)
