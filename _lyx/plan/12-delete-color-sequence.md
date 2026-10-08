# Card 12 — Delete the raw color sequence the verified send replaced

**Delete:**
- `internal/shuttleengine/claudeengine#Claude.ColorSequence`
- `internal/testkit/shuttlefake#Engine.ColorSequence`

**Edit:**
- `internal/shuttleengine#Engine`
- `internal/testkit/shuttlefake#Engine`
- `internal/shuttleengine/fakes_test.go`

**Intent:**
Delete the color key choreography card 11 left without callers, so the color reaches a session only through the verified send.
`Engine` drops its `ColorSequence` method and doc comment; `Claude.ColorSequence` goes from `color.go`, whose file header then names only the palette mapping and the command; `shuttlefake.Engine` drops `ColorSequenceFn` and its method; `fakes_test.go`'s `fakeEngine` drops its `ColorSequence`.

Before deleting, grep the repository for `ColorSequence`; any reference left outside the deleted code (a doc comment, a doc file) is reworded in this card.

**ImpactSummary:** Removes one interface method and its three implementations, none with a caller after card 11, so the build and every behaviour stay unchanged.

**Commit:** 12: shuttle, claudeengine: delete the raw color sequence the verified send replaced
