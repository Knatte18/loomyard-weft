# Card 13 — segment-colour-merge

**Create:**
- `plan:internal/segmentcolor#Merge` -> `const Merge Segment = "merge"`

**Edit:**
- `internal/segmentcolor#Segments`
- `internal/segmentcolor#TestClosedSets`
- `internal/reedengine/template_posix.yaml`
- `internal/reedengine/template_windows.yaml`
- `internal/burlerengine#Profile`
- `internal/burlerengine#Profile.reviewTable`
- `internal/burlerengine#Profile.fixTable`
- `internal/burlerengine#TestEngine_Review_TableShape`
- `internal/burlerengine#TestEngine_Fix_TableShapeAndFiles`
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#Bouncer.awaitLiveJudge`
- `internal/shedadapters#Bouncer.awaitLiveSeed`
- `internal/shedadapters#Bouncer.runSeedSpawn`
- `internal/shedadapters#Bouncer.judgeCall`
- `internal/shedadapters#TestBouncer_PromptComposition`

**Intent:**
Give the Merge-Review segment a colour of its own (D2, `PATTERN-spawn-color`), told on the launch spec and never asked of an agent.
`segmentcolor` gains `Merge`, last in `Segments()`, and both reed templates gain `merge: red` under `segment_colors`, a palette colour no other segment of a loom run uses.
`burlerengine.Profile` gains `Segment segmentcolor.Segment`; `reviewTable` and `fixTable` use it, and the zero value keeps `segmentcolor.Review`.
`shedadapters.BouncerConfig` gains `ColorSegment segmentcolor.Segment`; the seed and judge spawns and their live-session probes use it, and the zero value keeps `segmentcolor.Review`.
`TestClosedSets` pins the new segment, the two table-shape tests assert a profile with `Segment: segmentcolor.Merge` spawns under it, and `TestBouncer_PromptComposition` asserts the same for `ColorSegment`.

**ImpactSummary:** one more segment name and two optional colour fields; every existing row keeps the review colour.

**Commit:** 13: segmentcolor: a merge segment for the merge review's sessions
