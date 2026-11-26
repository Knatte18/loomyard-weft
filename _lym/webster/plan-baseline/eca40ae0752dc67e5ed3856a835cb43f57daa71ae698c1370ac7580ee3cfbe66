# Card 11 — module-key-messages

**Edit:**
- `internal/landingshed#`
- `internal/dobbyengine#`
- `internal/driftengine#`
- `internal/darnengine#`
- `internal/hardenshed#`
- `internal/shuttleengine#`
- `internal/shuttleengine/claudeengine#`
- `internal/shuttleengine/codexengine#`
- `internal/shedrecipe#`
- `internal/shedbuild#`
- `internal/runprofile#`
- `internal/gateslot#`
- `internal/orchengine#`
- `internal/websterengine#`
- `internal/heddlecli#`
- `internal/configengine#selectorWayForward`
- `internal/configengine#selectorRefusal`
- `internal/configengine#overlayTemplate`
- `internal/seatengine#TestEngine_Run_StartFailureEndingTheRunStopsEveryStartedSeat`
- `contracts/stencils#`

**Uses:**
- `plan:internal/configengine#KeyName`
- `internal/configengine/doc.go`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestFullEnv_PathFieldsAbsoluteUnderOneRoot`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/testkit/envkit#TestSeamMarkers_ReflectiveTestsAreMarked`

**Intent:** Every other module's Go message, refusal, way forward and halt notice that names a config key names it in full, `<module>.<dotted path>`, rendered through `configengine.KeyName` from the module's name constant and the key's path constant (D3).
The sites the exploration found: `landingshed`'s `beam.pull_request` and `landing.squash` texts and its `landing config key %q` refusal, and its comment naming `loom.pr.review_parent_timeout_min`; `dobbyengine`'s `beam config key %q` refusals and its carry lines naming `beam.pull_request`; `driftengine`'s refusals naming its keys, `drift.judge_ask_parent` included; `darnengine`'s `darn config key %q` refusals; `hardenshed`'s `hardener config key %q` refusals and its hub-only way forward; `shuttleengine`'s `ModelKey` refusal, `claudeengine`'s batching nudge and `codexengine`'s `sandbox_mode`; `shedrecipe`'s implementer and `shedbuild`'s entry-gate refusals, which name the referenced setting; `runprofile`'s profile refusal; `gateslot`'s tier and build refusals; `orchengine`'s reinterpretation line; `websterengine`'s `rename_tools` refusal; `heddlecli`'s override key; configengine's overlay selector refusals; and the comments in `contracts/stencils`' Go code naming `tapestry.parts.depth`.
The writer greps every module's Go code outside the packages card 10 owns for `config key %q`, `--set ` and each module's bare key names, and every further message holding a bare key is part of this card.
Each key path such a message cites becomes one named constant in its module where it is a literal today, and the module name a named constant where it is one.
A command a way forward prints keeps the CLI's own form, such as `lym config beam --set pull_request=merge`.
`shuttleengine`'s `ModelKey` refusal keeps its tail, `to an alias whose engine meets them`, so only the key name in it changes; `seatengine`'s asserting row moves to `set loom.review.reviewer_model ...`-style full names.
Each test that asserts one of these messages asserts the new text; it is a member of the packages listed or the seatengine member listed.

**ImpactSummary:** Reworded texts only, in about fifteen modules' refusals and notices and one seatengine assertion; no behaviour, value or CLI form changes.
