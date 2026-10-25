# Card 17 — land-drops-ready

**Edit:**
- `internal/battenshed#LandDeps`
- `internal/battenshed#NewLand`
- `internal/battenshed#landProducer.Call`
- `internal/battenshed#runMarkerSuffixes`
- `internal/battenshed#landFixture`
- `internal/battenshed#newLandFixture`
- `internal/battenshed#landFixture.deps`
- `internal/battenshed#TestLand_Dispositions`
- `internal/battenshed#TestLand_FailingStepsAreStuckWithOneNotice`
- `internal/battenshed#TestClearRunMarkers_RemovesEveryRunMarkerSuffixOnly`
- `internal/battencli#landDeps`
- `internal/shedrecipe#landEntry`
- `internal/testkit/envkit#nilLegal`
- `internal/testkit/envkit#FullEnv`
- `internal/battenshed/doc.go`
- `docs/overview.md`
- `//contracts/recipes/batten-recipe.yaml`

**Delete:**
- `internal/battenshed#ReadyRoute`
- `internal/battenshed#readySnapshotFileSuffix`
- `internal/battenshed#readyMarkerLine`
- `internal/battenshed#readyRouteMarkerPrefix`
- `internal/battenshed#landProducer.snapshotPath`
- `internal/battenshed#landProducer.newlyReady`
- `internal/battenshed#rowNotifier.ready`
- `internal/battenshed#landFixture.readyRoutes`
- `internal/battenshed#TestLand_ReadyNotice`
- `internal/battenshed#TestLand_ReadyRouteResume`
- `internal/battencli#readySlugs`
- `internal/battencli#readyRoutes`
- `internal/battencli#landReadyRoutes`
- `internal/battencli#notifyOrchAt`
- `internal/battencli#landingLineReachesPrime`
- `internal/battencli#TestReadySlugs`
- `internal/battencli#TestReadyRoutes`

**Create:**
- `internal/battenshed#rowNotifier.recordsStopped`

**Intent:**
Drop the landing notice's ready part and the `landed` notice itself (D6, D9): a landing that succeeds needs nothing from the orch, and dobby computes readiness from the board.
`LandDeps` loses `ReadyTasks` and `ReadyRoutes`, `landProducer.Call` loses the ready snapshot and the newly-ready read, and the row's last step after the board prune is the records-stop notice: `recordsStopped` posts one line, `<slug>: landed; the prime's records did not pull: <stop>`, once per batten run under its row-notice marker line, only when the pull's records side stopped, since that stop needs the orch's hand and no other notice carries it; a clean landing posts nothing.
`runMarkerSuffixes` loses the snapshot suffix; `NewLand`'s doc names the rows' steps without the ready notice.
`battencli.landDeps`, `shedrecipe.landEntry`'s seam list and `envkit`'s `nilLegal` and `FullEnv` lose the two seams, and batten's ready helpers go with their tests.
`doc.go`'s Land paragraph and `docs/overview.md`'s Land sentence drop the ready snapshot, the routes and the landing notice, and say a records stop is the one notice of a successful landing.
The recipe file's header comment, whose Land paragraph says Land "names the tasks the landing made ready", drops those words and states that a records stop is the one notice of a successful landing; card 13 edits the same header for Run-Shed only.
Tests: `TestLand_Dispositions` and `TestLand_FailingStepsAreStuckWithOneNotice` lose their ready seams and rows; a row asserts a clean landing posts no notice and a landing whose pull reports a records stop posts the one records-stop line once across a resumed row; `TestClearRunMarkers_RemovesEveryRunMarkerSuffixOnly` loses the snapshot suffix.

**ImpactSummary:** The Land row and its one wiring site lose two seams; shedrecipe's validation, envkit's fixtures and two battencli helpers with their tests go with them, and no other reader used the ready snapshot.
