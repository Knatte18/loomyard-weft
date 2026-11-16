```yaml
lens: affected-set
outcome: reported
```

Lens: affected-set. For each output, key, file shape or behaviour a card changes, I list everything that reads or asserts it and which cards cover that set.

**Per change: the set found, and the cards that cover it**

1. **Card 8: `judgeStatus` counts a finished `cancelled` record as a candidate.**
   - Readers: `cleanupSlug` and `judgeRunDirectory` (`cleanup.go:515`, `:525`), and `clearDoneRun` (`cleanup.go:222`), which `ClearDoneRun` and `ClearMotifRun` both call.
   - Card 8 covers only cleanup's own path. `clearDoneRun` is missed.
2. **Card 7: a kept cancel leaves a batten record whose shed state is `cancelled`, instead of no record.**
   - Readers that judge it by "state != done": the dobby's slot count `dobbyengine/view.go:105-106` and the run list `passes.go:184`; `beamcli/create.go:195-202`; `adoptcli/adopt.go:320` and `:536`; `battencli/startguard.go:490`; `dobbyreads.go:366`; `readoptstop.go:303`; `pairteardown/followup.go:35`.
   - Readers that switch over the state: `arm.go:531` and `:903`, `drive.go:167`, `claim.go:54`, `halt.go:148`, `resume.go:72`, `innerrun.go:572`.
   - Card 1 covers only the switches, through its grep for `case shedengine.StateDone`. Card 7 covers only `ClearDoneRun`. Every `== StateDone` / `!= StateDone` reader is missed.
3. **Card 6: the cancel marker file goes, replaced by the intent plus the local lock.**
   - Readers: `dobbyreads.go:187`, `pairteardown/teardown.go:196`, and `readCancelMarker`'s users in `startguard.go:87`, `:282` and `:457` and in `cancelcmd.go:251`. Card 6 covers these.
   - Doc and help text that describe the marker file: `cancelcmd.go:67` ("leaves its marker"), `battencli/cli.go:23`, `pairteardown/doc.go:154`, the comment at `teardown.go:713`, and `dobbyengine/doc.go:154`, `:286`, `:338` and `:385`. No card covers them.
4. **Card 7: the cancel's output and behaviour change (it keeps the record, writes a finish and adds a finish field).**
   - Readers: the renderer `cancelEnvelope` at `cancelcmd.go:150-175`; the help text `cancelCmd` Long at `cancelcmd.go:54-78`, which says the cancel "removes the run's records" and that "lym beam create &lt;slug&gt; starts the run again"; and the caller `fronttask/hub.go:67`.
   - Card 7 covers `cancelReport` and `cancelRun` only.
5. **Card 11: the pull-request record moves to `RunDir` and gains a `Publish` seam.**
   - Readers: `pawlwiring.go:175` (card 11); `prcontinue.go:78` (card 11); `landingdeps.go:348` and `:432` (card 11); `paths_test.go:77-80` (card 11).
   - Also: the fixture builder `internal/testkit/envkit#LandingDeps` at `envkit.go:183-225`; `pawlwiring_integration_test.go:88`; `commitrecords_test.go:197`; `pawlengine/retiredmove_test.go:57`; `pullrequest/doc.go:4`; `pairteardown/doc.go:101-102`; the comment at `teardown.go:406-408`.
   - Card 11 lists only envkit's tests, under `Uses`. The builder `LandingDeps`, the docs, and the tests outside its list are missed.
6. **Card 12: `landingshed.Deps` gains `AttemptsDir`, which `NewFinalize` and `NewParentAlign` refuse when empty.**
   - Readers: `loomcli#landingDeps` (card 12); the landingshed fixtures (card 12); `envkit#LandingDeps` at `envkit.go:183-192`, which fills `ScratchDir` only.
   - Card 12 misses envkit's builder, so every user of `envkit.LandingDeps` hits the new empty-dir refusal.
7. **Card 5: `RunView` gains `Records`, and the check on `CancelMarker` goes.**
   - Readers of `RunView.CancelMarker`: `runnotice.go:182`, `ended.go:37` and `passes.go:203`. Card 5 covers only `planPass`. The other readers keep working, since `DobbyRuns` still fills the field (card 6).
8. **Card 8: cleanup's help text.** `cleanupCmd` Long at `cleanup.go:628-637` says cleanup removes "every done run". No card covers it.

**Findings**

1. **BLOCKING · consistency · card 8 (`internal/battencli#judgeStatus`).** Making `judgeStatus` accept a finished `cancelled` record also makes `clearDoneRun` (`cleanup.go:222`) take one. `ClearDoneRun`, the dobby's done-run clear, would then remove a kept cancel's record and discard its task branch on origin. That contradicts D3 and card 7's claim that `ClearDoneRun` "takes only done runs". Card 8 must keep the cancelled candidacy out of `clearDoneRun`, for example with a separate result, and list `internal/battencli#clearDoneRun`.
2. **MEDIUM · design · card 1 (method; card 7 makes it bite).** Card 1's grep finds only `case` switches, so it misses every reader that compares against `StateDone` with `==` or `!=`. A kept cancelled record (card 7) then:
   - holds a `max_runs` slot forever (`dobbyengine/view.go:105`) and stays in the dobby's driven list (`passes.go:184`);
   - makes `lym beam create` refuse a cancelled beam (`beamcli/create.go:202`), although the cancel's help names it as the way forward;
   - reads to adopt as a live run (`adoptcli/adopt.go:320`, `:536`).
   Also missed: `startguard.go:490`, `dobbyreads.go:366`, `readoptstop.go:303` and `pairteardown/followup.go:35`.
3. **MEDIUM · consistency · cards 11 and 12 (`internal/testkit/envkit#LandingDeps`, `envkit.go:183-225`).** The shared landing fixture sets neither card 11's new `PullRequest.Publish` nor card 12's `AttemptsDir`. Its `RecordPath` still uses the scratch shape. Both cards list only envkit's tests, under `Uses`, so the builder's users fail (card 12's empty-dir refusal) or call a nil `Publish`.
4. **LOW · consistency · card 7 (`internal/battencli#cancelEnvelope`, `cancelCmd`).** The new finish field never reaches the JSON envelope (`cancelcmd.go:150-175`). The help text (`cancelcmd.go:54-78`) still says the cancel removes the run's records, leaves its marker, and that `lym beam create` restarts a beam. Neither is listed.
5. **LOW · consistency · card 11 (`internal/pullrequest/doc.go`, `internal/pairteardown/doc.go`).**
   - `pullrequest/doc.go:4` says the record "lives in the run's scratch directory, so it is never tracked". Card 11 must rewrite that line, but lists the file under `Extends`.
   - `pairteardown/doc.go:101-102` and the comment at `teardown.go:406-408` say "the record under the run's scratch directory". No card lists them.
   - The tests `pawlwiring_integration_test.go:88` and `commitrecords_test.go:197` read the record path and are not named.
6. **LOW · consistency · card 6 (docs of the cancel marker).** These still describe a marker file: `cancelcmd.go:67`, `battencli/cli.go:23`, `pairteardown/doc.go:154`, and `dobbyengine/doc.go:154`, `:286`, `:338` and `:385`. Card 6 edits only the `shedrun` and `runfsm` docs.
7. **NIT · consistency · card 8 (`cleanupCmd`, `cleanup.go:628-637`).** The help text still says cleanup removes only done runs.

Tool calls:
1. Grep for cancel-marker readers and `battencli.Cancel` callers repo-wide: the manifest held only the files I had read, not every reader.
2. Grep of envkit's landing fixture, the pull-request record path and fixer-attempt mentions: envkit's builder and the test readers were outside the manifest.
3. Grep for `StateDone` and `runStateDone` readers and the cancel's output keys: the readers that compare instead of switch were outside the manifest.
