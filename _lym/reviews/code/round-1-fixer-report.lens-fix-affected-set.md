```yaml
lens: fix-affected-set
outcome: reported
```

Task: check, through the affected-set lens, that each fix covers every member of its finding's set.

**Findings**

1. LOW — `internal/dobbyengine/state.go:40`, the `State.Land` field comment.
   - It reads "the landing step taken in the episode now standing, a land request asked for or a notice posted, nil when none was taken…".
   - F1 added a state where `Land` is set (`Refused: true`) yet the step stays open and Land is retried. The comment still presents any non-nil value as a step already taken.
   - It is a sibling of the `LandState` doc and still reads as the pre-F1 behavior.

2. LOW — `internal/dobbyengine/doc.go:112-113`, the bullet that opens the landing step.
   - It says a passing reading "with no episode recorded in State.Land" asks for the landing, and that "the episode is recorded once the notice goes out".
   - After F1, a refusal notice records a `Refused` episode that is still retried. Line 114, which the fixer added, covers this. Lines 112-113 are not qualified, so a reader meets two statements of when the episode closes in adjacent lines.

3. LOW — `internal/dobbyengine/doc.go:116`, the "Bound" bullet.
   - "a request it refuses is not asked for again in the same episode" is ambiguous after F1.
   - A request Beam-Await refuses is still not re-asked.
   - Land's own refusal is now re-asked each tick.
   - The line should say which of the two it means.

**Searches that found nothing stale**

- The F6/F7 wording search for "can land" notice, `land_notice` and "not asked for again" found no other site except the `doc.go` line above.
- The F3 `MarkReady` search found its seam in `landDeps` (`land.go:31`), the real dep (`deps.go:123`) and the test fake (`land_test.go:79`), and no other implementer or caller.
  - `markReady` has two callers, `cli.go:586` and the new integration test.
  - Both match the new always-call behavior.
- The ready-record wording search found no other doc that describes `lym beam ready` as writing nothing for an already flagged beam, so none went stale.
- Finding F5 (the `pawlRecheck` ready read) is left to F2's beam-level test, as the review allowed. That test covers `hubLandDeps.Ready` and `beamLandReading`, not `pawlRecheck`'s own read, so the direct read stays unpinned.

**Unchecked:** F2 and F4 premises are about missing rows, not sets I could re-search within the cap, so I did not re-run a search for them.

**Tool calls**
- grep for stale landing-notice wording, `MarkReady` and `LandState` consumers across `*.go` and `*.md`: the context held only the files the fixer read, not the repo-wide sites.
- grep for ready-record wording in docs, plus `sed -n 111,118p internal/dobbyengine/doc.go`: the context lacked the doc text after the fixer's edit and any other doc sites.
