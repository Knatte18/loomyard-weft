```yaml
lens: fix-consistency
outcome: reported
```

Fix-consistency lens: checking that the change set agrees with its sibling cards, decisions, criteria and rows.

**[MEDIUM] Card 6, failing-LoadConfig row (F12 fix).** The row now reads "a failing `LoadConfig` posts nothing and the check posts once it loads again;". The text that said it calls `d.gateHolds` directly is gone, so the row now runs through the harness's probe ticks like its siblings, and those ticks cannot carry it.
- `startReady` returns `load beam.yaml: %w` when `LoadConfig` fails (`internal/dobbyengine/starts.go:93-95`). `tick` passes that error up (`internal/dobbyengine/loop.go:720-722`).
- `passHarness.tick` calls `t.Fatalf("tick() = %v; want nil", err)` on any error that `tick` returns (`internal/dobbyengine/passes_test.go:245-247`).
- So the reason the old text gave held for the harness. `gateHolds` is reached before `startReady`, but the row's tick still fails the test. The review's premise covers only the order of the steps.

**[MEDIUM] Card 3, Entry upkeep, the new sentence that sets the `LandNotice` copy to nil (F3 fix).** The sentence reads "An entry whose slug is not listed this tick and that `State.LandNotices` holds no memory of has its `LandNotice` copy set to nil". It also fires after the landing notice has already posted, which breaks D3's skip and the acceptance criterion "none when it is on".
- `settleLandNotices` drops the memory once the line posts (`internal/dobbyengine/landnotice.go:177`). D3 says the landing notice decides on the tick the run passes Land, "often a tick before" it derives as landed.
- The breaking sequence:
  1. On tick N the run is past Land but not yet landed. Its landing notice posts and the memory is dropped.
  2. On tick N+1 the run is unlisted, for example because its status is unreadable, so the copy goes nil.
  3. On tick N+2 the run is listed as landed. Its reading differs from `Last`, which held a non-landed reading. The copy is nil, so the landed reading "is never skipped" and a watch line posts beside the landing notice already sent.
- The condition needs to tell "the landing memory was dropped unposted" apart from "dropped after posting". For example, clear the copy only when `Last` shows the run still ahead of Land.

**[LOW] Card 3, the new `clearDoneRuns` hold "while `RunWatch.Level` is set, a run with no level read yet" (F1 fix).** This hold disagrees with the card's Seams sentence: "A nil `Level` or a nil `Post` sends no watch line and keeps no memory".
- A harness or wiring that sets `Level` but leaves `Post` nil holds the clear on a `watchLevels` entry. Nothing says the watch step still reads the level in that case, so the run could be held forever.
- The hold should depend on both seams being set, or say that the level is read whenever `Level` is set.

**[LOW] Card 3, ImpactSummary.** It still says "`clearDoneRuns` gains one hold". After the F1 and F11 fixes it gains three: a watch line still owed, a level not yet read, and a tick whose watch step failed to record (`watchRecordFailed`). The doc.go sentence in the same card lists all three.

Tool calls:
- grep of `_lym/plan` and the decision record for `TestCheckLocalValues` and "one hold", plus `passHarness.tick`'s error handling: my context lacked both the stale-reference sweep and the harness code.
- grep of `internal/dobbyengine` for `LoadConfig` callers: needed to confirm that `startReady` is what fails the tick.
- read of `internal/dobbyengine/starts.go:85-100`: needed `startReady`'s handling of a failed `LoadConfig`, which my context did not show.
