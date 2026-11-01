```yaml
lens: affected-set
outcome: reported
```

Lens: affected-set. For each change the plan makes, I list everything that reads or asserts it and the cards that cover it.

**Per change: the set and the cards that cover it**

1. Adopt envelope renames `suite`, `message` and `suite_warning` to `checks`, `message` and `check_warnings` (card 5).
   - Set: `adoptcli/cli.go` `adoptFields` (cli.go:122,128) and its `Long` text (cli.go:49); `adoptcli/cli_test.go` `TestAdoptFields` (:37,:46); `adoptcli/adopt_test.go` `order` "suite" entries (:196,:601,:626,:677); `adoptcli/doc.go` (:80,:120).
   - Covered by card 5. Cards 11 and 14 also edit `TestAdopt`. No reader outside `adoptcli`.
2. `ReadCheckView` is removed, and `adopt_check` becomes `adopt_checks` (card 5).
   - Set: `beamcli/cli.go:51,136,140` (`statusResult`, `dobbyStatus`) and the `statusCmd` help (cli.go:78); `beamcli/doc.go:101`; `beamcli/status_integration_test.go:129-148` (`TestStatus_PrintsThePrimesDobbyAndEachLocalBeams`); `adoptcli/doc.go:114`; `docs/overview.md:457`.
   - Covered by card 5.
3. The check verb becomes `adopt check <subject>`, with per-subject files (card 5).
   - Set: `suiteArgs` and `startSuite` (deps.go); `TestCheckCmd_FromABeamExitsZeroWhileOneRuns` (adopt_integration_test.go:203); `fronttask`'s action row (card 6); `pattern/PATTERN-test-speed.md`; `docs/overview.md:447`.
   - Covered by cards 5 and 6.
   - `reedengine/bar_integration_test.go` passes literal mark strings to `SetCheckMark`, so the subject prefix does not reach it. Nothing is missed.
4. Adopt no longer resumes runs; the green check resumes them (cards 11 and 12).
   - Set:
     - `adopt.go` strings that say adopt resumes: `heldBeamWayForward` (adopt.go:670, "takes the parent over and resumes the landing here"), `ownerWriteError` (adopt.go:564,566, "so nothing was resumed"), the `write.Lost` error (adopt.go:306) and the read-back error (adopt.go:314).
     - `landingshed/finalize_owner.go:117`, the Finalize owner refusal: "adopt it here with "%s", which resumes the halted run".
     - `beamcli/doc.go:31`: "only this verb, a hand pass or `lym adopt` starts or resumes its run".
     - `adoptcli/doc.go` and `docs/overview.md`.
   - Covered: card 11 lists `adopt`, `adoptcli/doc.go` and `docs/overview.md`.
   - Not covered: `heldBeamWayForward`, `ownerWriteError`, `landingshed/finalize_owner.go`, and `beamcli/doc.go`'s line 31 (no card's Intent changes that line).
5. A new required darn marker path (card 8).
   - Set: `loomshed/darnwrite_test.go:52` (`newDarnFixture`); `darnengine/spec_test.go:24` (`newSpecInputs`); `loomcli/darnwiring.go:159,237` (`darnDeps`, `wireDarn`); `shedrecipe/entries_darnwrite_test.go:41` (`darnTestEnv`); `envkit.go:428` (`FullEnv`); the stencil.
   - Covered by card 8. `loomrecipe/darnrecipe_test.go:28` builds an empty `DarnDeps{}` only for its reflect type, so it is unaffected.
6. The board gains `front` and `on_land` in the upsert keys and Use token (card 3).
   - Set: `boardcli/cli.go:479` Use token and :506 help, which `TestPayloadUse_MatchesDeclaredKeys` reads; `boardengine` doc; `plugins/ly/skills/board/SKILL.md`, which lists no payload keys.
   - Covered by card 3.
7. `RunView.Held`, `TaskView.Front` and `TaskView.Project`, and `WaitingTask.FrontFirst` (cards 2 and 7).
   - Readers: `battencli/dobbyreads.go`, `overviewdobby.go` and `beamcli` deps. Covered by cards 2 and 7.
   - Whether a reflective seam test must cover these fields belongs to the repo-invariants lens.

**Findings**

1. **MEDIUM, consistency.** Location: card 11 (`adopt-holds`) and card 12 (`check-green-end`). The resume set is covered only in part: four texts that still say adopt resumes are left unlisted.
   - Card 11 moves the resumes out of adopt, and card 12 puts them at the check's green end.
   - Neither card lists `internal/landingshed/finalize_owner.go`. Finalize's owner refusal (finalize_owner.go:117) still tells the operator that `lym adopt` "resumes the halted run". After this plan, adopt resumes nothing; the subject's green check resumes the run, and only once the check is green.
   - Card 11's `adopt` edit does not name three helpers in `adopt.go`, and its Intent does not reword them:
     - `heldBeamWayForward` (adopt.go:670): "`lym adopt %s` takes the parent over and resumes the landing here".
     - `ownerWriteError` (adopt.go:564,566): "so nothing was resumed".
     - The lost-owner error (adopt.go:306) and the read-back error (adopt.go:314), which sit inside `adopt` and say "nothing was resumed".
   - `internal/beamcli/doc.go:31` ("only this verb, a hand pass or `lym adopt` starts or resumes its run") becomes false. Cards 5 and 7 list `beamcli/doc.go`, but neither Intent changes this line.
   - Fix: add `internal/landingshed#finalizeOwner…` (the function holding finalize_owner.go:117), `internal/adoptcli#heldBeamWayForward` and `internal/adoptcli#ownerWriteError` to card 11, and rewrite each to name the subject's adopt check. Extend card 11's Intent to the in-`adopt` error texts and to `beamcli/doc.go`'s line 31.
   - Evidence: internal/landingshed/finalize_owner.go:117; internal/adoptcli/adopt.go:306,314,564,566,670; internal/beamcli/doc.go:31.

No other set is partly covered: changes 1–3 and 5–7 are fully covered by the cards named above.

Tool calls:
1. Grep for the envelope keys, `adopt_check` and `ReadCheckView` across the repo, because the manifest held only the `adoptcli` and `beamcli` definitions, not their readers.
2. Grep for files outside `adoptcli` that mention `lym adopt` or `adopt check`, and for the locations of the cited test functions, to find readers the manifest lacked.
3. Grep those files for resume, check and scratch wording, to see which texts the move of the resumes falsifies.
4. Grep for the `DarnInputs`, `DarnDeps` and `StopMarkerPath` construction sites, and for plugins and contracts that mention adopt, to find the fixtures the new marker forces.
5. Grep the board skill and the orch role stencil for adopt mentions and payload keys, to check docs outside the manifest.
6. Grep the board skill for a payload-key listing, to settle whether it must name `front` and `on_land`.
