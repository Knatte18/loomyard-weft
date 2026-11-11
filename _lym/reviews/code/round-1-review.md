---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/adoptcli/worker.go:226
    summary: "A resume that restarts at reconcile clears the recorded steps, so the end envelope loses the earlier process's pause and pairs parts."
    fix: "In runWorker, clear record.Steps and record.StoppedPart only for verdictFresh, or when mergeRequest reports restart. Otherwise keep the done steps and let recordStep replace each as it re-runs. Add a TestRunWorker row where a resume restarts at reconcile after pairs and pause were done, and assert the envelope still holds the earlier pause part (AdoptPaused, paused run state, PauseStanding) and the pairs mutations."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/adoptcli/steps.go:318
    summary: "The stop at done names a re-run of the held-back subjects only, which judgeStanding refuses as other subjects of the standing record."
    fix: "In stepDone build keep and drop from run.Req (same slugs, root and all) with only KeepLocal or DropLocal changed, so the named command passes SameSubjects against the stopped record and resumes at reconcile. beamcli/adoptsection.go reads record.WayForward and needs no change. Add a test that feeds the stop's own way forward command back through judgeStanding."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/adoptcli/steprun.go:247
    summary: "Nothing pins that a stored StoppedPart is cleared when the step later succeeds and then leaves the envelope."
    fix: "Add a TestRunWorker row in internal/adoptcli/worker_test.go: stop at a step that gathered a part, resume so the step succeeds, then assert Record.StoppedPart is empty and the end envelope holds the step's part once."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/adoptcli/worker.go:214
    summary: "A resume re-resolves --all against the live board instead of the recorded subjects, so run.Slugs can differ from record.Subjects."
    fix: "On verdictResume take slugs and rootNamed from record.Subjects (the word root becomes rootNamed) instead of calling namedEntries again. Keep the namedEntries call for a fresh record."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/pauserecord/doc.go:25
    summary: "OwnerEntryExempt and the parent-lock release key on OwnerCaller only, so a slug whose owner entry D5 converts to an adopt entry loses both, and the doc is silent on it."
    fix: "Decide and state it. Either add to internal/pauserecord/doc.go that the parent-lock exemption and the parent-lock release deliberately ignore a converted adopt entry, or extend OwnerEntryExempt (pauserecord/hold.go) and the release in battenshed/innerrun_parentlock.go to AdoptCaller."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/dobbyengine/owned_test.go:143
    summary: "No dobby test shows that the owner lift leaves an adopt or adopt-hold pause entry standing (D5 acceptance)."
    fix: "Add a row to the owner-lift test in internal/dobbyengine/owned_test.go with an AdoptCaller entry covering an owned-elsewhere run, asserting the entry stays after the lift."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/tendercli/cli_test.go:138
    summary: "No test pins that lym tender pause on an adopt-paused slug answers already_paused true (D5 acceptance)."
    fix: "Add a row to the pause test in internal/tendercli/cli_test.go that seeds an AdoptCaller entry for the slug, runs the pause, and asserts already_paused is true and the entry is unchanged."
  - id: F8
    severity: LOW
    class: consistency
    origin: handler
    location: internal/adoptcli/worker.go:270
    summary: "reconcileWire drops the report's embedded mutations, so the envelope's reconcile object always carries empty mutations, where the pre-change envelope carried them."
    fix: "Add Mutations fabricengine.Mutations with json tag mutations to reconcileWire and copy it into the OriginReport's MutationRecord in decodePart, or state in the adoptcli doc.go envelope description that reconcile.mutations is empty and the top-level mutations field holds the record. Add an assertion in the worker envelope test."
  - id: F9
    severity: LOW
    class: consistency
    origin: handler
    location: internal/adoptcli/cli_test.go:13
    summary: "TestAdoptFields is contained by TestAdopt in coverage, but only it pins the envelope map's key presence and omission rules, which TestAdopt never reads."
    fix: "Keep the test and add a //testtiming:keep line directly above func TestAdoptFields, with a reason naming what it pins: which envelope keys appear only when set and that runs is always present."
  - id: F10
    severity: LOW
    class: consistency
    origin: handler
    location: internal/adoptcli/holds_test.go:13
    summary: "TestTriesStore is contained by TestFailedEnd in coverage, but only it pins that reading an absent tries file creates no directory."
    fix: "Keep the test and add a //testtiming:keep line directly above func TestTriesStore, with a reason naming what it pins: an absent file reads zero and a read creates no directory."
---

### [MEDIUM] Resume at reconcile clears the earlier steps

**Location:** `internal/adoptcli/worker.go:226`
**Issue:** D2 says a resuming worker keeps the done steps and stored inputs, and `recordStep` already replaces a step of the same name. `runWorker` instead sets `record.Steps` and `record.StoppedPart` to nil whenever `from` is `reconcile`.
After such a resume the end envelope is merged from the new process's steps only.
The `pairs` mutations disappear, so `mutations` and `partial` under-report.
The earlier `pause` step's slug fields (`AdoptPaused`, the paused run state, `PauseStanding`) disappear too, and the re-run `pause` step skips slugs already in `Inputs.Paused`, so nothing restores them.

### [MEDIUM] The held-back way forward is refused by the standing record

**Location:** `internal/adoptcli/steps.go:318`
**Issue:** `stepDone` names `lym adopt <held-back projects> --keep-local` and the `--drop-local` variant. The stopped record keeps the whole adopt's subjects, or `--all`.
`judgeStanding` compares with `Request.SameSubjects`, so the named command is refused as other subjects unless the held-back set equals the record's.
`lym beam status` shows the same command, since it prints `record.WayForward`.

### [MEDIUM] No test pins the clearing of a stored stopped part

**Location:** `internal/adoptcli/steprun.go:247`
**Issue:** The Added-after-Discussion entry requires the stopped part to be cleared when the step later succeeds. `recordStep` does so, but no test fails if it stops. The only `StoppedPart` reference in tests is a round-trip in `internal/adoptrecord/record_test.go:40`.

### [LOW] A resume re-resolves --all against the live board

**Location:** `internal/adoptcli/worker.go:214`
**Issue:** D2 records the resolved subject set. On resume `namedEntries` runs again, so a beam created or landed since the interruption changes `run.Slugs`, while `stepCheck` and `refuseBeforeSteps` use `run.Slugs` and the envelope uses `record.Subjects`.

### [LOW] A converted owner entry loses the parent-lock exemption and release

**Location:** `internal/pauserecord/doc.go:25`
**Issue:** `OwnerEntryExempt` (`pauserecord/hold.go`, used at `tendercli/gate.go:84`) and the release in `battenshed/innerrun_parentlock.go:36` key on `OwnerCaller`. D5 converts an own-scope owner entry into an adopt entry in place, so a converted slug no longer gets either. Neither the diff nor the doc says whether that is intended.

### [LOW] No dobby test for the owner lift leaving an adopt pause

**Location:** `internal/dobbyengine/owned_test.go:143`
**Issue:** The D5 acceptance criterion that the dobby's owner lift leaves every adopt pause and legacy `adopt-hold` entry standing has no test row. The existing `Held` rows set the field directly and never see the caller values.

### [LOW] No test for already_paused on an adopt-paused slug

**Location:** `internal/tendercli/cli_test.go:138`
**Issue:** The D5 acceptance criterion that `lym tender pause` on an adopt-paused slug answers `already_paused: true` has no row. The existing test repeats only a plain pause.

### [LOW] The envelope's reconcile object loses its mutations

**Location:** `internal/adoptcli/worker.go:270`
**Issue:** `OriginReport` embeds `MutationRecord`, which serializes as `mutations`. `reconcileWire` leaves it out, and `Save` always builds the envelope through `envelopeOf`, so the written envelope's `reconcile.mutations` is empty. The top-level `mutations` field holds the entries, but the nested copy the old envelope carried is gone and nothing says so.

### [LOW] TestAdoptFields needs a keep marker, not a fold

**Location:** `internal/adoptcli/cli_test.go:13`
**Issue:** The redundancy report lists it as contained, covered by `TestAdopt`. `TestAdopt` asserts the `outcome` struct (`adopt_test.go:888-984`), never the map `adoptFields` builds, so only this test pins which envelope keys appear only when set.

### [LOW] TestTriesStore needs a keep marker, not a fold

**Location:** `internal/adoptcli/holds_test.go:13`
**Issue:** The redundancy report lists it as contained, covered by `TestFailedEnd`. That test checks the count only (`fixloop_test.go:161`); the no-directory-created assertion on a read of an absent file is covered nowhere else.

## Rejected

- Lens code-tests, `spawnWorker` has no test for detach, registration and the spawn log: the function returns early under `testing.Testing()`, as the repo's other spawn sites do, and it wraps `proc.Detach`, `runproc.Register` and `logger`, which have their own tests.
- Lens code-tests, an undecodable record turning `beam status` into a failure has no test: the lens did not check the integration test, and the failure names `lym adopt --discard` as its way forward.
- Lens code-correctness, the follow's last sleep can overshoot the keep-warm cap by up to eight seconds: the lens judged it harmless, and the cap is a soft bound on a cold cache.
- Lens code-tests, the green end's negative assertions (no lift, no run start) cannot fail: the seams were removed from `greenDeps`, so the behavior cannot regress without a compile error.
