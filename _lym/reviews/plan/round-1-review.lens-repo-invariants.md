```yaml
lens: repo-invariants
outcome: reported
```

**Repo-invariants lens: each card checked for test seams and test-file tiers, falsified doc and PATTERN text, and whether it can pass its own gate after the cards before it.**

## Per-card list

| Card | Test seams and tiers | Doc and PATTERN text | Card order and own gate |
|---|---|---|---|
| 1 | held: the new package's untagged `testmain_test.go`; `TestReadWrite` is tier0 | held: `adoptcli/doc.go` names the file, not the functions | held: every reference to `readTries`/`writeTries` is in a listed test (`TestGreenEnd` at `green_test.go:149`, `TestRunCheck`, the `fixFixture` helpers) |
| 2 | held: tier0 | held: `doc.go` is edited | held |
| 3 | held | held | held |
| 4 | held: tier0 and tier1 are stated per test | held: `machinecli/doc.go` is edited | held |
| 5 | held | held | held |
| 6 | held | held | held |
| 7 | held | held | held: the re-signed callers are left to the gate's `caller-uncovered` check |
| 8 | held | held | **broken**, finding F1 |
| 9 | held | held | held |
| 10 | held | **broken by a later card**, finding F3 (card 23 makes the `recoveries` text false) | held |
| 11 | held | **broken**, finding F5 (PATTERN-cli-cobra) | held |
| 12 | held | the doc is edited, but card 23 later makes it false (finding F2) | held |
| 13 | held: `TestAct` and `TestResume` cover the reworded refusal (`act_test.go:38`, `resume_test.go:419`) | held | held |
| 14 | held: tier1 | held | held |
| 15 | held | held | held |
| 16 | held | held | held |
| 17 | held: tier1 | held | held |
| 18 | held | held | held |
| 19 | held: tier1 | held | held |
| 20 | held | held: `shedrun/doc.go` never names the marker | held: every `OriginOwed` reference is in a listed test or deleted (`owner_test.go:272` `TestOwnerPreflight_TeardownResume`, `run_test.go:431` `TestRunCmd_ResumeDispositions`, `owner_readopt_test.go:477` `TestOwnerPreflight_GoneTaskWorktree`, `continue_pass_test.go:107` `TestBattenPreRun_ContinuePass`, `passes_test.go:430` `TestDrivePasses_ExitHandling`, the deleted `originowed_test.go`) |
| 21 | held | held | held: deleting `pauseRunsForOwner` breaks no test, though `runAnAdoptHold` still sets `PauseOwned` until card 23 |
| 22 | held: the allowlist gains `internal/fixtries` in this card | held | held |
| 23 | held: the `InnerRunDeps` seam tests and envkit's `nilLegal` are listed | **broken**, findings F2, F4 and F6 | **broken**, finding F1 |
| 24 | held: `TestOwnedElsewhereEnder` and the `endOwnedElsewhere` stub (`cli_test.go:332`) are deleted with the code | held | held |
| 25 | n/a | **partly broken**, finding F6 | held |

## Findings

**F1 — BLOCKING, design, cards 8 and 23: two cards re-shape `dobbyengine` seams without listing every test that builds them, so their packages' tests do not compile.**
- Card 8 changes `Deps.Owner` to return `(owner string, elsewhere bool, err error)`.
  It lists only `newLoopHarness` and `TestRun_ForeignOwner` among the tests that set it.
- These tests also assign `h.deps.Owner = func() (string, error)` and no card lists them:
  - `runAnAdoptHold` (`handover_test.go:163`)
  - `TestRun_ALaterArchiveFailureLogsAgainAfterTheOwnerLeaves` (`handover_test.go:289`)
  - `TestRun_AFailedBoltPullLeavesTheOwnerReadToTheLocalBolt` (`:321`)
  - `TestRun_BoltPullBringsInANewOwner` (`:352`)
  - `TestRun_LockOwnerAndWakeups` (`loop_test.go:210/229/242`)
  - `TestRun_TickFailuresLogOncePerEpisode` (`loop_test.go:697`)
- The `PrimeOwner` reshape is filled in `fakeStart.baseDeps` (`start_test.go:66`), a method, while card 8 lists only the `fakeStart` type.
- Card 23 removes `Deps.PauseOwned`, `Deps.LiftOwnerPause` and `Deps.Machine`.
  These still set them, and card 23 lists neither:
  - `newPassHarness` (`passes_test.go:223/229/239`)
  - `runAnAdoptHold` (`handover_test.go:178`)
- Fix: list those test members under `Edit` on cards 8 and 23.

**F2 — MEDIUM, consistency, card 23: `internal/pauserecord/doc.go:17-28` becomes false and no card lists it.**
- The doc says `OwnerCaller` entries are paused with "its lifter".
- It says "A dobby's owner-pause lift and the hibernate wake judge only `OwnerCaller`".
- Card 23 retires both the lift and the wake's owner pause.
- Card 12 edits this doc, but it comes before card 23 and cannot drop text that is still true at that point.
- Fix: list `internal/pauserecord/doc.go` under `Edit` on card 23.

**F3 — MEDIUM, consistency, card 23: the user-facing `recoveries` text in `internal/machinecli/set.go:35` becomes false and no card edits it.**
- The text reads "A run paused for an owner other than this machine resumes once the sweep records this machine as owner: the dobby's probe lifts that owner pause."
- Card 23 deletes both the owner pause and the probe lift (`recheckOwnerPause`, `liftOwnerPause`).
- Card 10 edits `machinecli` before that and keeps the text.
- Fix: card 23 edits `internal/machinecli#recoveries`.

**F4 — LOW, consistency, card 23: `pattern/PATTERN-dobby-runs.md:21` names "the dobby's owner lift", which card 23 retires, and no card lists the file.**
- The rule says the adopt pause lifts "never the adopt check, the dobby's owner lift or a hibernate wake".

**F5 — LOW, consistency, card 11: `pattern/PATTERN-cli-cobra.md:64` becomes false and no card lists it.**
- The background says `machinecli` calls only `machinename`, `boardengine`, `fabricengine` and `hubregistry`.
- Card 11's `hubReferences` calls `pairteardown.ReadBattenRecord`.
- Fix: card 11 either edits that line or reads the record through a package the line already admits.

**F6 — LOW, consistency, cards 25 and 23: text in `pattern/PATTERN-owner-machine.md` and `adoptcli/doc.go` stays describing the retired owner pause.**
- Card 25's named edits leave in `PATTERN-owner-machine.md` the line "An entry owned elsewhere is paused under the owner, naming `lym adopt`" (line 21).
- They also leave the Enforcement line "They cover `InnerRun`'s owner pause" (line 78).
- Card 23 retires both behaviours; the card says only that "its sections change with the code".
- `internal/adoptcli/doc.go:33` ("go on to Land under its owner pause") and `:210` ("an owner pause included") become stale the same way, and no card edits them.

## Tool calls
1. Grepped the `//lym:seam` markers and envkit's `nilLegal`, to find which shared structs reflective tests walk; my context had no seam markers.
2. Grepped `PATTERN.md` and `pattern/` for retired terms, to find background text the cards make false; my context had only some background files.
3. Read the `PATTERN-dobby-runs` and `PATTERN-cli-cobra` spans those hits pointed to, which were not in my context.
4. Grepped the `doc.go` files the cards list under `Uses`, for falsified text; they were unread.
5. Mapped the test lines that reference deleted seams to their enclosing functions, to check them against each card's `Edit` list; my context had only the line numbers.
6. Mapped the `dobbyengine` tests that set the owner seams card 8 re-shapes, which were unread.
