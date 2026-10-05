# Card 7 — Bouncer judge and seed load their skill and render the parent directive

**Edit:**
- `internal/shedadapters/bouncer.go`
- `internal/shedadapters/doc.go`
- `internal/shedrecipe/recipe.go`
- `internal/shedrecipe/entries_bouncer.go`
- `internal/loomcli/wiring.go`
- `contracts/stencils/bouncer/bouncer-template-judge.md`
- `contracts/stencils/bouncer/bouncer-template-seed.md`
- `internal/shedadapters/bouncer_judge_test.go`
- `internal/shedadapters/bouncer_seed_test.go`
- `internal/shedrecipe/entries_bouncer_test.go`

**Uses:**
- `plan:internal/parentdirective#Directive`
- `plan:internal/parentdirective#MarkerName`
- `internal/shedadapters#BouncerConfig`
- `internal/shedrecipe#Env`

**Intent:**
Move the Bouncer's two spawned sessions onto D1 and D9.

`shedrecipe.Env` gains `ParentName`, a run-wide value of exactly the class Env carries; the Bouncer entry passes it to a new `BouncerConfig.ParentName`.
Loom's `wire()` sets `Env.ParentName` from the receiver's `parentName` field (card 4); batten's Env builds no Bouncer row and leaves it empty.

Skills (D2), declared beside `bouncerJudgeRole` and `bouncerSeedRole`: `scribe:prose` for both, set on the judge and seed spawn specs (`judgeCall`'s and `runSeedSpawn`'s).
The attach probes `awaitLiveJudge` and `awaitLiveSeed` build specs only to find a live run, and are unchanged.

The judge and seed renders fill `parentdirective.MarkerName` with `parentdirective.Directive(b.cfg.StencilsDir, b.cfg.ParentName, false)`.
`bouncer-template-judge.md` and `bouncer-template-seed.md` carry `{{.parent_directive}}` near their top.
The escalation brief and parent notice are read by the parent's own fork and by the parent, not by a spawned role, and carry none.

Tests: the judge and seed prompts carry the directive with the parent's name and the no-parent variant without one, and their spawn specs carry `scribe:prose`; the Bouncer entry passes `Env.ParentName` through to the config.
`doc.go` names the skill and the directive in its Bouncer section.

**ImpactSummary:** two Bouncer spawn renders, one config field and one Env field; loom's wiring is the one production Env writer that sets it.

**Commit:** 7: bouncer: judge and seed load their skill and render the parent directive
