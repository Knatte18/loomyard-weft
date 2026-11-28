---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/dobbyengine/starts.go:362
    summary: A first pass whose own catch-up fails while the dobby's catch-up succeeds, or on a dobby with no catch-up wired, is started again every tick or two without bound, with no backoff, no notice and a full pass owed to every run each cycle.
    fix: "In firstPassExited's PassExitOriginWait case, read the exit as owing the catch-up only when the catch-up can answer it: call d.oweCatchUp(), and if d.catchUpOwed is still false (Deps.CatchUp has no PullBolt) fall through to d.refusedStart(slug, f.fingerprint, text, now) as the default case does. Also bound the repeat: keep a per-slug record (in StartState, e.g. a field set on an origin-wait exit and cleared by the claimed and exit-0 cases) that the start's last exit was PassExitOriginWait; when a first pass exits PassExitOriginWait again after the dobby's catch-up has succeeded since that exit (catchUpSucceeded ran in between, which a counter or timestamp on the daemon set in catchUpSucceeded can tell), read it as a refused start through d.refusedStart, so it gets the notice and the doubling backoff. Add rows to TestStartReady_RefusalsAndBackoff: (a) the pass keeps exiting PassExitOriginWait while fakeCatchUpDeps succeeds, and the second such exit posts one notice and sets a start backoff, so no third pass spawns before NextTry; (b) Deps.CatchUp left empty, and an origin-wait first pass records a refused start with a backoff. Correct the change description's Design decisions bullet that says no case loses its report."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-scope
    location: internal/dobbyengine/pushsummary.go:95
    summary: Only the prime's own catch-up wait reaches the push summary, so a beam dobby whose start waits on origin all night is reported to no one unless the prime's catch-up happens to be owed too, contrary to the goal's report past the bound and to doc.go:271 and the change description.
    fix: "Add a seam to PushSummaryDeps that lists the catch-up waits of every dobby of the hub, e.g. CatchUpWaits func() (map[string]CatchUpWait, error) keyed by worktree name, wired in internal/beamcli/dobbydeps.go to read each beam dobby's state file and the prime's own. In summarisePushes, replace the read of record.CatchUp with a loop over that seam's waits whose Reason is CatchUpWaitOrigin, calling earliest(wait.waitingSince()) and appending \"<worktree>'s catch-up from origin for <n>m\". A nil seam reads the prime's own record as now. Add a TestPushSummary row where only a beam's wait is listed and is summarised past the threshold. Then make doc.go:271 and the change description's third Design decisions bullet state that the prime's summary reports every dobby's catch-up waiting on origin."
  - id: F3
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: internal/battencli/drive.go:138
    summary: "The goal licenses treating an unreachable origin as a transient stop, but PassExitOriginWait also carries a refused registration and every other catch-up failure, so those first passes lose their refused-start notice and backoff too."
    fix: "In passExitCode, map a catchUpRefusal to dobbyengine.PassExitOriginWait only when errors.Is(err, dobbyengine.ErrOriginUnreachable); map any other catchUpRefusal to a new constant dobbyengine.PassExitCatchUpRefused declared in internal/dobbyengine/geometry.go beside PassExitOriginWait. In passExited, list PassExitCatchUpRefused with PassExitOriginWait in the no-failure case and owe the catch-up for it, so a later pass keeps today's behaviour. In firstPassExited, leave PassExitCatchUpRefused to the default case, a refused start. Update the catchUpRefusal and passExitCode doc comments, doc.go:73-74 so it names only a catch-up that could not reach origin, and the change description's first Design decisions bullet. Add a TestStartReady_RefusalsAndBackoff row where a PassExitCatchUpRefused first pass gets one notice and a backoff."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/dobbyengine/doc.go:281
    summary: The package doc's list of what the summary notice names covers owed branches, refused ones and waiting runs, but not the dobby's catch-up from origin the diff adds to it.
    fix: Extend the sentence at doc.go:281 to also list the dobby's catch-up waiting on origin, with its minutes.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/dobbyengine/pushsummary_test.go:111
    summary: The fallback of waitingSince to At for a record written without Since, a behaviour the change description states, has no test.
    fix: "Add a TestPushSummary subtest that seeds the state record through UpdateState with CatchUp set to CatchUpWait{Reason: CatchUpWaitOrigin, At: h.start} and no Since, then checks no notice at 9m and a notice containing \"the dobby's catch-up from origin for 10m\" at 10m."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/dobbyengine/loop_catchup_test.go:191
    summary: Nothing pins that Since restarts when the failure reason changes; dropping the reason comparison in catchUpFailed passes every test.
    fix: "Add a row to the catch-up test that holds the rows at loop_catchup_test.go:191 and :221: a catch-up fails with a non-origin error at t0, then with ErrOriginUnreachable at t1 and t2, and the state record's CatchUp.Since equals t1."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/dobbyengine/pushsummary_test.go:111
    summary: No test checks that a catch-up waiting as refused or failed stays out of the push summary; removing the Reason == CatchUpWaitOrigin filter passes every test.
    fix: Add a TestPushSummary subtest in which the dobby's catch-up fails with a boardengine.RegistrationError past the threshold and the push summary posts no notice.
  - id: F8
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/dobbyengine/pushsummary.go:137
    summary: "The dobby's catch-up line is posted under the heading \"runs waiting on the network:\", but the catch-up is no run."
    fix: "Change the heading at pushsummary.go:137 to \"waiting on the network:\", or post the catch-up line under its own part, such as \"the dobby's catch-up waits on origin: ...\"."
---

### [BLOCKING] An origin-wait first pass the dobby's catch-up cannot settle restarts without bound

**Location:** `internal/dobbyengine/starts.go:362`

**Issue:** The pass's own catch-up covers only its slug (`internal/battencli/wire.go:1043`).
The dobby's catch-up covers `DobbyPairSlugs`, the pairs that stand, and a task not yet started has none (`internal/beamcli/dobbydeps.go:324`, `internal/battencli/catchup.go:340`).
So a failure tied to the slug never shows up in the dobby's catch-up.
Examples are a failed `FetchRecords`, which `fetchRunForCatchUp` reads as an unreachable origin (`internal/battencli/catchup.go:242`), a `ReadRun` error, a failed `PushOwed`, or `ErrRecordsLayout`.
On such a failure, tick N reads exit 80 and owes the catch-up.
On tick N+1 the dobby's catch-up succeeds, clears the owed flag and sets `owesAll` (`internal/dobbyengine/catchupwait.go:114`, `:125`).
`startReady` then finds no start record and spawns the pass again (`internal/dobbyengine/starts.go:139-153`).
This repeats every one or two ticks, with no backoff, no notice and no wait record, and each cycle owes every run a full pass.
On a dobby whose `Deps.CatchUp` has no `PullBolt`, `oweCatchUp` does nothing (`internal/dobbyengine/catchupwait.go:51`), so the next tick spawns the pass at once.
Before the diff, both cases got a notice and a backoff of up to an hour.
The change description's claim that "no case loses its report" does not hold for either case.
The goal asks for a retry on the dobby's backoff that is reported past a bound, and this path has neither.

### [MEDIUM] A beam dobby's catch-up waiting on origin reaches no one

**Location:** `internal/dobbyengine/pushsummary.go:95`

**Issue:** `summarisePushes` reads only the state file of the dobby it runs in (`pushsummary.go:86`), and only the prime's dobby has a summary (`pushsummary.go:40`).
A beam dobby's `CatchUpWait` is read only by its view (`view.go:108`).
The prime's catch-up becomes owed only on a sleep, a dobby start, a hand-over, or an exit of its own pass (`loop.go:510`, `:712`; `catchupwait.go:173`; `owned.go:21`; `passes.go:468`; `starts.go:363`).
So during an outage that none of those triggers, a beam's start waits silently all night, which is the situation the board entry's Seen describes.
The change description claims that the prime's dobby, "waiting for the same reason, reports the episode", and nothing in the code makes the prime wait.
The `doc.go:271` sentence "the push summary reports it past its threshold" is false for every beam dobby.

### [MEDIUM] Refused and failed catch-ups lose their start refusal

**Location:** `internal/battencli/drive.go:138`

**Issue:** The goal names "a catch-up fetch on an unreachable origin" (`_lym/discussion/decision-record.md:11`).
`passExitCode` maps every `catchUpRefusal` to `PassExitOriginWait`.
`catchUpFailure` builds a `catchUpRefusal` for an unreachable origin, a `RegistrationError` and any other failure alike (`internal/battencli/catchup.go:64-80`).
So the new `firstPassExited` branch also drops the refused-start notice and backoff for a refused registration and for every other catch-up failure.
The change description records this widening as the writer's own design decision, and no entry in the record licenses it.
It also widens F1's loop to every failure of the pass's catch-up.

### [LOW] The package doc's notice contents omit the catch-up line

**Location:** `internal/dobbyengine/doc.go:281`

**Issue:** The doc says the notice lists each owed branch, each refused one with its way forward, and each waiting run with its producer.
It does not list the catch-up line the diff adds at `pushsummary.go:98`.

### [LOW] The fallback to At for a record without Since is untested

**Location:** `internal/dobbyengine/pushsummary_test.go:111`

**Issue:** Every write since the change sets `Since`, and no test reads a record without it through the summary.
`waitingSince` could return the zero time and every test would still pass.

### [LOW] The reset of Since on a change of reason is untested

**Location:** `internal/dobbyengine/loop_catchup_test.go:191`

**Issue:** Without the `s.CatchUp.Reason == reason` check at `catchupwait.go:85`, an origin wait would inherit the start of an earlier refused or failed episode, and no test would fail.

### [LOW] The origin-only filter in the summary is untested

**Location:** `internal/dobbyengine/pushsummary_test.go:111`

**Issue:** Refused and failed catch-ups post their own notice and are meant to stay out of the summary.
No row would fail if the filter at `pushsummary.go:95` were removed.

### [NIT] The catch-up line sits under a heading for runs

**Location:** `internal/dobbyengine/pushsummary.go:137`

**Issue:** The catch-up line is appended to `waiting` and posted after "runs waiting on the network:", but the dobby's catch-up is no run.

## Rejected

- code-correctness: the `startReady` guard skipping `holdReadyBeam` and the `busy` update for one tick. The lens itself judged this harmless, since the ready-beam notice is only one tick late and `busy` was already set.
- code-correctness finding 2, the missing `PullBolt` seam leading to an unbounded respawn, is folded into F1. It is the same defect, and F1's fix settles it.
- code-tests: the untested `PullBolt`-nil branch is folded into F1, whose fix adds that row.
- code-affected-set finding 1, the false `doc.go:271` sentence, is folded into F2, whose fix corrects it.
