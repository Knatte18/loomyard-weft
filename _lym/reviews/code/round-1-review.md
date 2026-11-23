---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    origin: lens:code-scope
    location: internal/landingshed/finalize_pr.go:524
    summary: The board entry's Finalize case, a merged landing whose parent upstream fetch failed on a transient network error, still halts at once as a landing stop with no retry and no network notice.
    fix: "In landingshed's Finalize, where ReadParentTips(true) fails after the merge (finalize_pr.go:524-525), retry the fetch with backoff over the same bounded schedule while gitexec.IsTransportFailure(err) holds, through an injectable sleep like the PR-merge loop's pr.Sleep above it; only once the retries are spent halt with LandingStopPrefix, and word the halt reason as a notice naming the failing fetch. Add a landingshed test row that fails the parent-tips read with a transport error twice, then succeeds, and asserts the landing goes on with no halt, and one that stays failing and asserts the halt."
  - id: F2
    severity: BLOCKING
    class: scope
    origin: lens:code-scope
    location: internal/battencli/resumereconcile.go:33
    summary: "The pre-run and catch-up origin reconciles still make one ls-remote attempt, so the entry's run that stopped on git ls-remote origin at its start, and #203's refused hand pass, still reach the parent with no backoff; the summary defers this with no recorded parent decision."
    fix: "Pass a ReconcileOptions with a production RetryBound and RetryInterval (at least a minute in total, as hibernatecli/reconcile.go does with its wake bounds) at the run-start reconcile sites internal/battencli/resumereconcile.go:33, internal/battencli/readopt.go:142 and internal/beamcli/dobbydeps.go:456, taking the bound from a field the CLI's tests can set short so their refused-origin fixtures do not wait it out; add a battencli test that scripts one transport failure then success and asserts the run resumes."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/gitexec/transport.go:36
    summary: "The new phrases unexpected disconnect while reading sideband packet, send failure and early eof also appear in permanent refusals, such as a push GitHub rejects with HTTP 413 as too large, so such a refusal is now re-stepped over 1m45s and reported to the parent as a network failure."
    fix: "In IsTransportFailure, return false when the stderr holds an HTTP 4xx status line (\"the requested url returned error: 4\" or \"rpc failed; http 4\") before the phrase loop runs, so a 4xx refusal wins over a disconnect phrase printed after it; add TestIsTransportFailure rows with a 413 push stderr carrying the sideband disconnect and a Send failure line, each expecting false."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/sheddriver/drive.go:982
    summary: The real-clock branch of goDriver.wait, the only one production runs, and settleStop's exit on a wait error have no test; every TestGoDriver_Settle row wires a Wait stub that returns nil.
    fix: "Add a TestGoDriver_Settle row (or a field on its table) whose Wait stub returns context.Canceled at the first network-failure stop, asserting the driver returns that error with no re-step, settle record or hand-back; and add a sheddriver test that leaves Deps.Wait nil, drives a network-failure stop with an already-cancelled context, and asserts the drive ends with context.Canceled without waiting the delay."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/sheddriver/drive_test.go:311
    summary: "No test pins that the wait comes before the pause hold and the status recheck, which the summary claims so that a pause or orch goto made during the wait is honoured; moving the wait after recheck passes every test."
    fix: "Add a TestGoDriver_Settle row with a network-failure stop whose Wait stub moves the scripted status by an orch goto, asserting a moved settle record and a re-step at the new row rather than a re-step record at the failed stop."
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/sheddriver/doc.go:62
    summary: The package doc's seam list and pattern/PATTERN-shed-driver.md omit the new optional Deps.Wait and still say an unwired optional seam only removes rows, which a nil Wait does not.
    fix: "In internal/sheddriver/doc.go's \"Seams a recipe fills\" section and pattern/PATTERN-shed-driver.md's optional-closure list, add `Wait` and state that an unwired Wait waits on the real clock rather than removing a row."
  - id: F7
    severity: LOW
    class: scope
    origin: lens:code-scope
    location: internal/fabricengine/pushanchored.go:350
    summary: "#203 names weft pushes of the board and a records branch failing on the same network drop, each with its own notice; the diff touches no weft push path and leaves their backoff unverified."
    fix: "site not found — find the weft push paths for the board and a records branch that call gitexec.IsTransportFailure (pushanchored.go:350 and pushclassified.go:332, 536, 635); where a transport failure is pushed once and then reported owed, retry it with backoff over the same bounded schedule through an injectable sleep before it is reported, and add a fabricengine test row with one transport failure then success."
  - id: F8
    severity: NIT
    class: design
    origin: lens:code-tests
    location: internal/gitexec/transport_test.go:49
    summary: The sideband row's stderr holds both new phrases, so deleting either phrase from transportPhrases alone passes the test.
    fix: "Split the row \"fetch-pack: unexpected disconnect while reading sideband packet\\nfatal: early EOF\" into two rows, \"fetch-pack: unexpected disconnect while reading sideband packet\" and \"fatal: early EOF\"."
---

### [BLOCKING] Finalize's parent upstream fetch gets no retry

**Location:** `internal/landingshed/finalize_pr.go:524`

**Issue:** The board entry's second case is `models-wemod-robustness-sanity`, blocked at Finalize because the parent's upstream could not be fetched after the PR merged.
`ReadParentTips(true)` failing halts with `LandingStopPrefix`; the stop is a blocked hand-back stop, and `decideDrive`'s landing-halt row (`internal/sheddriver/driverules.go:271`) matches before the new network rows.
So this stop still goes straight to the parent with no wait, re-step or network notice.
The entry's Expected asks that a transient failure on a git fetch be retried with backoff before the run stops.
The summary does not list this case under "Not covered".

### [BLOCKING] Run-start and catch-up reconciles still try ls-remote once

**Location:** `internal/battencli/resumereconcile.go:33`

**Issue:** `reconcileRun`, `readopt.go:142` and dobby's `dobbydeps.go:456` pass a zero `ReconcileOptions{}`, and `attempts()` (`internal/fabricengine/originreconcile.go:83`) returns 1 for it.
The entry's third case stopped at the start of its run on `git ls-remote origin`, and #203's hand pass was refused with "origin is unreachable" (`internal/battencli/catchup.go:66`); both still happen on one failed call.
Both Expected sections ask for backoff over a minute or more before a run halts.
The summary's reason for deferring, that integration tests would wait out a production bound, is answerable: `ReconcileOptions` already carries `Sleep`, `RetryBound` and `RetryInterval` seams, which `hibernatecli` sets.
The record holds no added entry that licenses the deferral.

### [MEDIUM] New transport phrases also match permanent refusals

**Location:** `internal/gitexec/transport.go:36`

**Issue:** A push GitHub refuses as too large prints `error: RPC failed; HTTP 413` followed by `send-pack: unexpected disconnect while reading sideband packet`, and can print `Send failure: Broken pipe`.
`IsTransportFailure` matches any listed substring, so this refusal now classifies as `git-transport`.
The driver then re-steps it three times over 1m45s and tells the parent "a network failure outlasted 3 re-steps", and `pushclassified.go` reads it as an owed push rather than a refusal.
The 403 and 404 rows pass only because those lines carry no disconnect phrase.

### [MEDIUM] The production wait path and the wait-error exit are untested

**Location:** `internal/sheddriver/drive.go:982`

**Issue:** Neither `battencli/drive.go:257` nor `loomcli/drivewiring.go:236` sets `Deps.Wait`, so production runs only the timer branch of `goDriver.wait`.
Its stated contract, returning the context's error when the context ends, has no test, and neither does `settleStop`'s return at `drive.go:289-293` when the wait fails.
The mutation report's uncovered changed blocks fit these branches.

### [LOW] The wait's order before pause and recheck is not pinned

**Location:** `internal/sheddriver/drive_test.go:311`

**Issue:** The summary claims a pause or orch goto made during the wait is honoured before the re-step, because the wait runs before `holdPause` and `recheck`.
No row combines a network-failure wait with a pause or goto, so moving the wait after `recheck` would pass every test.

### [LOW] Seam docs omit Deps.Wait

**Location:** `internal/sheddriver/doc.go:62`

**Issue:** The package doc's "Seams a recipe fills" list and `pattern/PATTERN-shed-driver.md` name every optional seam and say an unwired one only removes rows.
`Wait` is missing from both, and a nil `Wait` removes no row: it waits on the real clock.

### [LOW] Weft pushes are left to the existing owed-push path

**Location:** `internal/fabricengine/pushanchored.go:350`

**Issue:** #203 reports weft pushes of the board and a records branch failing on the same drop, each with its own notice.
The diff only widens the classification these paths read; it adds no backoff to them and no test shows the existing path retries before reporting.

### [NIT] The sideband test row pins two phrases at once

**Location:** `internal/gitexec/transport_test.go:49`

**Issue:** One stderr carries both `unexpected disconnect while reading sideband packet` and `early eof`, so deleting either phrase alone still passes.

## Rejected

- code-correctness finding 2, the network notice dropping the stop report path: a relay hand-back already sends only its relay line under `driveNotices`' existing design, as the awaiting parent-notice relay does, and the notice carries the failing operation the entry asks for.
- code-affected-set candidate 5, `FinalizePastMerge` reading a recorded re-step: a tackler re-step at a transient Finalize already wrote the same `SettleRestep` record before this diff, and the reader acts only on an owned-elsewhere takeover or a cancel, so the diff adds no new state; no defect was shown.
- The second commit's Root-Cause trailer and the notice wording were checked against `driverules.go` and `networkNotice` and hold; no finding.
