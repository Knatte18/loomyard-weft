# Card 10 — strand-provider-tag

**Create:**
- `plan:internal/shuttleengines#SessionNamers` -> `func SessionNamers() map[string]reedengine.SessionNamer`
- `plan:internal/shuttleengines#namersOf` -> `func namersOf(engines map[string]shuttleengine.Engine) map[string]reedengine.SessionNamer`
- `plan:internal/shuttleengines#TestSessionNamers` -> `func TestSessionNamers(t *testing.T)`

**Delete:**
- `internal/shuttleengines#SessionNamer`

**Edit:**
- `internal/reedengine#Strand`
- `internal/reedengine#AddSpec`
- `internal/reedengine#Engine.addStrandLocked`
- `internal/reedengine#SessionNamer`
- `internal/reedengine#Engine.WatchNames` -> `func (e *Engine) WatchNames(ctx context.Context, namers map[string]SessionNamer) error`
- `internal/reedengine#Engine.watchNames` -> `func (e *Engine) watchNames(ctx context.Context, namers map[string]SessionNamer, t nameRepairTiming) error`
- `internal/reedengine#Engine.repairNames` -> `func (e *Engine) repairNames(namers map[string]SessionNamer) (repaired bool, err error)`
- `internal/reedengine#TestRepairNames_PaneTitle`
- `internal/reedengine#TestRepairNames_SessionName`
- `internal/reedengine#TestRepairNames_SkipsSessionCheckWithoutNamerOrSessionID`
- `internal/reedengine#TestRepairNames_UnanswerableSessionCheckRepairsNothing`
- `internal/reedengine#TestWatchNames_RepairedAnswerChoosesTheNextWait`
- `internal/reedengine#TestNaming`
- `internal/reedengine#TestCalleeFailuresLogOncePerEpisode`
- `internal/reedengine#TestAddStrandLocked`
- `internal/reedengine#TestSaveState_ThenLoadState_RoundTrips`
- `internal/reedengine#TestStrandOps_RealTmux`
- `internal/reedengine/doc.go`
- `internal/shuttleengine#Runner.start`
- `internal/shuttleengine#Runner.Open`
- `internal/shuttleengine#TestRunner_Start_HappyPath`
- `internal/shuttleengine#TestRunner_Open`
- `internal/testkit/shuttlefake#Reed.AddStrand`
- `internal/reedcli#enterSession`
- `internal/shuttleengines/doc.go`
- `pattern/PATTERN-shuttle-provider-seam.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/reedengine#fakeNamer`
- `internal/reedengine#repairStrand`
- `internal/reedengine#newRepairTestEngine`
- `plan:internal/shuttleengines#entry`

**Intent:**
reed strands carry an opaque provider tag that shuttle sets, and the watchdog repairs a session name through the namer for each strand's tag, so reed stays blind to what a tag means (D9).

`Strand` and `AddSpec` gain `Provider string`, JSON `provider` on the strand, omitted when empty, so a strand recorded before the field decodes with none (`PATTERN-tolerant-records`); reed stores it and never interprets it.
`addStrandLocked` stamps `spec.Provider` onto the new strand, which covers `AddStrand`, `AddStrandUnless` and `ReplaceStrand`, whose new strand is built from its spec through the same helper; resume's relaunch rebinds the recorded strand in place and never rebuilds it, so the tag survives it with the rest of the record.
Shuttle sets `Provider` to the run's resolved engine name in `Runner.start` (`engineName`) and `Runner.Open` (`engineName`); shuttle's attach reconstructs a run over its recorded strand and rewrites no strand, and a respawn is a fresh `Runner.start`, so both keep the tag.
`shuttlefake`'s `Reed.AddStrand` copies `Provider` onto the strand it mints.

`WatchNames`, `watchNames` and `repairNames` take a map from provider tag to `SessionNamer` in place of one namer: `repairNames` looks up each strand's tag, and a strand whose tag has no namer, or a nil map, gets no session-name repair while its pane-title repair is unchanged.
`SessionNamer`'s comment says the provider implementations reach reed keyed by an opaque tag.
`shuttleengines.SessionNamers` replaces `SessionNamer`: it builds every registered engine through its constructor with a zero shuttle config and zero `Dirs`, which construction never reads beyond storing them, and returns `namersOf` over them, which maps each engine that implements `reedengine.SessionNamer` to its namer and skips one that does not, and also sends the empty tag to the `claude` engine's namer, by name and whatever `shuttle.yaml`'s default engine is, permanently, since strands recorded before the tag and strands added outside shuttle (`lyx reed add`) have always been repaired as Claude's.
Bound: the empty-tag entry repairs only a strand with a session id whose name has drifted, and a namer finding no Claude session for a non-Claude strand repairs nothing.
`reedcli`'s `enterSession` passes `shuttleengines.SessionNamers()`.

Tests:
- `TestRepairNames_SessionName`'s table gains three fields: `provider`, set on the strand `repairStrand` builds; `namers`, a map from tag to `fakeNamer`, replacing the single namer; and `title`, the live pane's title, defaulting to the strand's name.
  Its rows gain: a strand tagged `claude` under a map whose `claude` namer drifts and whose empty-tag namer does not, renamed once with the `claude` namer's text and its drift query recorded on that namer alone; an untagged strand under the reverse map, renamed through the empty tag's namer alone; and a strand tagged `codex`, with no `codex` namer and a title that differs from its name, typing no rename and querying no namer, while the row asserts the `select-pane -T` title repair was sent.
  So a lookup that ignored the tag fails the first two rows, and one that skipped the strand fails the third.
  The other listed tests pass a map where they passed one namer.
- `TestAddStrandLocked` gains a row whose spec carries a provider and asserts the strand carries it; `TestSaveState_ThenLoadState_RoundTrips` round-trips a strand's provider and decodes a strand recorded without the field to an empty provider.
- `TestStrandOps_RealTmux` adds its strands with a provider, gives its `ReplaceStrand` spec a provider and asserts the replacement carries it, and asserts the strand `Resume` relaunches keeps its provider, so the tag survives replace and resume's relaunch against a real tmux.
- `TestRunner_Start_HappyPath` and `TestRunner_Open` assert the recorded `AddSpec` carries the engine name; shuttle's attach rewrites no strand and a respawn is a fresh start, which these two rows cover.
- `TestSessionNamers` asserts the `claude` and empty tags both map to a `*claudeengine.Claude` and no other tag is present, and that `namersOf` over a fake engine set holding one engine that implements `SessionNamer` and one that does not maps only the first.
- `cmd/lyx/testdata/test-budget.yaml` raises `internal/shuttleengines` to 5.

Docs: `reedengine`'s `doc.go` describes the opaque tag and the per-tag namers; `shuttleengines`' `doc.go` drops its "`SessionNamer` lives here for now" line for `SessionNamers`; `PATTERN-shuttle-provider-seam`'s background says reed's provider tag is opaque and the namers reach reed through `shuttleengines`.

**ImpactSummary:** One new optional strand field and a re-signed watchdog entry point with one production caller (`reedcli`'s watchdog); every existing strand keeps today's Claude name repair through the empty tag.
