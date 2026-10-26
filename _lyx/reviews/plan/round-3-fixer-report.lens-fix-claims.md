```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: I checked every sentence the fix added or rewrote against the code at HEAD.

**LOW**
- Changed text: card 16's new Intent sentence, "`nilLegal` lists `Pawl.Watch` since a nil watch leaves the timer alone".
- Summary: the reason holds, but the entry may do nothing, because `NilSeams` may never report this seam.
- Evidence: card 15:43 calls `Watch` "the request directory's change channel".
  `envkit.walkNil` (`internal/testkit/envkit/envkit.go`, after line 489) reports only these nil kinds: Func, Interface, Pointer, Map, and a Slice of Func.
  It never checks a Chan.
  So if `Watch` is typed as a channel, `Pawl.Watch` never appears in `NilSeams`, and its `nilLegal` row is dead.
  If `Watch` is a func that returns a channel, the row is needed.
  The card doesn't say which type, so the sentence is only right for the func-typed reading.

The rest of the change set holds:
- **Card 10, `prime` repoint:** `beam start` resolves at HEAD as an operator-audience verb (`internal/beamcli/cli.go:144`).
  The pinned rows sit at `internal/shortcutcli/shortcut_test.go:218` and `cmd/lyx/shortcuts_test.go:124`.
- **Card 10, smoke helpers:** `startPlainRun` (lines 474, 539), `killTreeProcs` (471, 535) and `tailString` (573) are called only inside `TestSmokeOrch_Adopt` (463–578).
  `waitForDiag` is called at 399, inside `TestSmokeOrch_CompactCycle`.
- **Card 14:** `TestLandingDeps_EveryFieldPopulated` fails on any zero field outside `unfilled`.
  That covers the string `ParentWorktree` as well as the func `LandBlockers`.
  `walkNil` reports a nil func, so `Landing.LandBlockers` needs its `nilLegal` row.
  With that row in place, neither wire test reports it, so "both wire tests keep passing" holds.
- **Card 16, `BeamDescribe` comparison:** both `intentionallyNil` tables list the `BeamDescribe` seams today: loomcli at `wiring_seams_test.go:79-81` and battencli at `wire_seams_test.go:40-42`.
- **Card 15:** `cmd/lyx/tiertags_test.go` demands exactly one tier tag on a test file holding Test functions, which agrees with the new wording.

Tool calls:
1. Read the loomcli `intentionallyNil` table, envkit's `nilLegal` and `walkNil`, the `unfilled` map in `TestLandingDeps_EveryFieldPopulated`, and the battencli wire test. My context had only their grep line numbers, not their contents.
2. Grepped loomcli's `BeamDescribe` rows and card 15's `Watch`/`DobbyLive` text. My context didn't show whether loomcli lists `BeamDescribe` or what type `Watch` is.
