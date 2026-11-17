```yaml
lens: affected-set
outcome: reported
```

Affected-set lens: what each card's change forces outside the card, and whether a card covers it.

**Per change, the set found and the cards that cover it**

- **Card 1, `PassageReader` re-signed and `StandingPassages` returning `Standing`.** Readers:
  - `claudeengine.go:262` and `codexengine.go:167` (`Claude.Prepare` and `Codex.Prepare`).
  - `cmd/lym/main.go:180`, plus the comment at line 179 (`newRoot`).
  - The reader installs in `claudeengine/prepare_test.go:596,644` and `codexengine/prepare_test.go:427`, both named `TestPrepare_AppendsStandingPassages`.
  - `cmd/lym/prerunseams_test.go:55`, which checks the install by name only, and the name is unchanged.
  - The `shuttleengine/doc.go:32` text.

  Card 1 covers all of them.
- **Card 1, `Renderer.Passages` and `passagesOf` re-signed.** Readers:
  - `render.go:362`, which is `Show`.
  - `render.go:388`, which is `Passages`.
  - `render_test.go:226` in `TestShowAndPassagesReadTheShippedLayerFirst`.
  - `render_test.go:754,758,762,793` in `TestRender`.

  Card 1 covers all of them.
- **Card 1, Codex item name `heddle passage N` becomes `heddle passage <slug>`.** Readers: `developerinstructions.go:49` and the `codexengine/prepare_test.go:415` row inside `TestPrepare_AppendsStandingPassages`. Card 1 covers both.
- **Card 1, `Engine` set on the `claude-file` and `codex-file` rows of `targetTable`.** The only readers of `Target.Engine` are `target.go:160` (`promptTargetsFor`, prompt rows only) and `overview.go:153` (`promptReach`, which only prompt-delivery paths call). No package outside `heddleengine` reads `Target`. The change needs nothing beyond card 1.
- **Card 2, `sweepOrphans` and `findRunByStrand` skip `manifests`.** Their readers are `TestSweepOrphans_AgeGuardAndLiveGuid`, `TestFindRunByStrand` and the `shuttleengine/doc.go` orphan-sweep text. Card 2 covers all three.
  - The card says the other root scans already skip a directory with no `run.json` silently. This holds: `carryover.go:146-148`, `stepstop.go:77-79`, `activity.go:95-103`, `agentwait.go:46-53`, `sessionread.go:53-60`, `enginesrun.go:68-70`, plus `attach.go` and `waitmark.go`, all `continue` on not-found.
- **Card 2, new fields on `UserSetup`, `InstructionItem` and `OmittedHook`.** The writers are `claudeengine` `UserSetup`, `skillListing` and `pluginSessionStartContexts` (card 3); the reader is `codexengine` (card 4). Covered.
- **Card 2, copy step in `Runner.launch`.** The fake engine writes no record, so no existing run test sees a new entry. Covered by card 2's own test.
- **Card 3, `skillListing` gains a result.** Its one caller is `usersetup.go:105`, which is `UserSetupSource.UserSetup`. Covered.
- **Card 5, `constructor` gains a parameter.** Affected sites:
  - Registry literals at `shuttleengines.go:59,81`.
  - Calls at lines 200, 228, 294, 307 and 319 (`SessionNamers`, `RecordReaders`, `ToolBoundaryRenderer`, `BatchingNudger`, `EditRecorder`).
  - `build`, called from `New` and `keepWarmWith`.
  - Test literals at `keepwarm_test.go:71` (`TestKeepWarm`) and `shuttleengines_test.go:136,145` (`TestNew`).

  Card 5 covers all of them.
- **Card 8, new `manifest` verb.** Affected: `Command` with its doc comment, `docs/overview.md` (shuttle verbs near line 640), and `TestCommand_PositionalArgValidation`. Card 8 covers them.
  - `resolve_verbs_test.go` is a two-row sample, not a complete list of verbs, so it needs no row.
  - No other shuttle test enumerates the verbs.

**Findings:** none. Every set found is covered by the card that makes the change.

**Tool calls**

1. Searched the callers of `PassageReader`, `Passages` and `passagesOf`, the `heddle passage` name, `skillListing` callers and `constructor` literals: the manifest named the definitions, not every caller and test.
2. Mapped those test lines to their enclosing test functions and searched docs for passage and verb text, to name the tests and doc readers.
3. Searched the pattern, docs and engine `doc.go` files for passage-reader and orphan-sweep wording, to find doc readers the manifest lacked.
4. Searched the verb-list tests and each run-root scan's handling of a missing `run.json`, to check card 2's claim and card 8's verb set.
5. Read `resolve_verbs_test.go` and the head of `TestCommand_PositionalArgValidation`, to see whether either enumerates every verb.
6. Searched for readers of `heddleengine.Target` outside the package, to check whether card 1's `Engine` change leaks into CLI output.
