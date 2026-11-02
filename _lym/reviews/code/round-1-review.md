---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: handler
    location: internal/hibernatecli/deps.go:113
    summary: "Expected bullet 2 (notice reaches the orch by its real creel name) is not made: the diff leaves receiver resolution through NotifyPrime untouched and only retries a refused notice, with no test of the resolution"
    fix: "In internal/hibernatecli/deps.go hubDeps, settle why the receiver came out as ly:orch (orchroute.StrandOf answers the fixed name <shortname>:orch when reed no longer holds the recorded GUID) and make the Notify seam resolve the orch strand's current creel name at each attempt, so the retry in sendNotice can reach it; add one test in internal/hibernatecli that pins the resolved name for a prime whose GUID reed no longer holds. When no orch strand is recorded NotifyPrime returns posted false with a nil error, which the seam drops: have the seam log that at Warn so the dropped notice is visible. If the retry is meant as the whole answer, the parent records that with lym loom decision add instead."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/hibernateengine/watch.go:166
    summary: "Two branches of the notice-file code have no test: a short multi-line line routed to the file, and a notice-file write failure that falls back to the log"
    fix: "In internal/hibernateengine/watch_test.go add two rows to TestWatch. First, a reconcile Stop under noticeLineBound containing a newline, asserting the queued line is one line within the bound and notice.txt holds the whole text. Second, a world whose NoticePath sits under a regular file, asserting the queued line is short and says the whole notice could not be written."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/hibernateengine/watch.go:142
    summary: "With no Owned in the request and a kept entry, reconcileAtWake judges the live hub entry as someone else's and skips the reconcile, then Watch lifts kept, so the notice says the pause stands and was lifted; no row reaches this path"
    fix: "In Watch, build target (r with Owned set to kept when r.Owned is nil) before calling reconcileAtWake and pass target to it, so the daemon's own entry is recognised and exempted; keep liftOwned(deps, target). Add one TestWatch row with a request that has no Owned and a kept entry, asserting the reconcile ran, the entry is lifted and the notice has no contradiction."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/hibernateengine/watch.go:154
    summary: "A finish(PhaseDone) write failure after a successful lift, and a fail notice with a long reason, have no test"
    fix: "In internal/hibernateengine/watch_test.go add two TestWatch rows. First, break the request file from the world's Reconcile hook, so advance has passed and finish fails, asserting the entry is lifted, the wake notice is sent and the write error is returned. Second, a long Reconcile error asserting the fail notice is the short headline and notice.txt holds the whole reason."
---

### [MEDIUM] Notice receiver resolution is unchanged

**Location:** internal/hibernatecli/deps.go:113, internal/orchcli/notify.go:133-139, internal/hibernateengine/watch.go:166-181
**Issue:** The board entry's second Expected bullet asks the wake notice to reach the orch by its real creel name, after a first failure naming receiver `ly:orch` as nonexistent. The diff's `hubDeps` still calls `orchcli.NotifyPrime`, whose `PrimeOrchStrand` answers the fixed name `<shortname>:orch` when reed no longer holds the recorded GUID. The only change is that `sendNotice` queues a refused notice again for up to ten minutes. That helps only if the name was right and the orch was not yet listed; the diff offers no evidence for that and no test pins the resolved name. Separately, `NotifyPrime` returns posted false with a nil error when no orch is recorded, and the seam drops that, so the loop never retries and the notice is lost without a Warn. The short-line half of the bullet is met.

### [MEDIUM] Notice-file branches without a test

**Location:** internal/hibernateengine/watch.go:166-169, 185-194
**Issue:** The newline condition of the file routing and the write-failure fallback are behavior the diff adds with no covering row. The existing over-bound row uses one long single-line text, and no row makes the write fail.

### [LOW] Wake with no recorded Owned and a kept entry gives a contradictory notice

**Location:** internal/hibernateengine/watch.go:142-153, internal/hibernateengine/wake.go:58
**Issue:** When `recordOwned` failed, `r.Owned` is nil while `kept` holds the entry. `reconcileAtWake` compares the live hub entry against `r.Owned`, sees no match, skips the reconcile and tells the operator to run `lym tender resume`. `Watch` then lifts `kept` and the headline says the pause was lifted. The lift is correct and no state is wrong; the reconcile does not run and the text contradicts itself.

### [LOW] Two wake failure paths untested

**Location:** internal/hibernateengine/watch.go:154-160, 298
**Issue:** The write failure of the done phase after a lift and the `fail` notice built from a long reason are both new behavior; only the earlier, pre-advance write failure has a row.

## Rejected

- Lift order, focus 2: the wake still reconciles before it lifts, but every failure path, including advance, reconcile, lift and notice, lifts through `fail` or the main path with `kept` as the fallback identity, a failed notice runs after the lift, and the retry holds no lock. Expected bullet 1 is met.
- `advance` returning `ok == false` without lifting `kept` (lens:code-correctness F2): only a cancel or a vanished request file reaches it, a cancel lifts the entry itself, and the behavior is unchanged by the diff.
- `Cancel` on a terminal request, focus 3: `stopPID` is set only inside the closure that returns `errUnchanged` for a terminal request, so no kill and no rewrite happen, an entry the request does not own is never named, and a request with no `Owned` reports `Lifted` nil; verbs_test.go covers both.
- `Lifted` non-nil with an empty `Resumed` for an already lifted entry (lens:code-correctness F4): matches the `CancelReport.Lifted` doc, which says nil only when the request owned none.
- Stale `Short` text and the long `Long` line in `cancelCmd` (lens:code-affected-set F-A): `Short` stays true for an ended request, and line wrapping is never a finding.
- Doc order of the terminal-request sentence before the `Bound:` line (lens:code-affected-set F-B): the diff places the new sentence above `Bound:`, and the fork read the diff only.
- Wiring test for `NoticePath` (lens:code-tests finding 6): the fork did not check the integration test, and the wiring is one line over an exported constant.
- Stop on a shared branch now lifting the hub entry as a scope excess: Expected bullet 1 licenses lifting on every wake path, and every stopped entry is in `Runs`, so each run is held first (`internal/hibernatecli/reconcile.go:58-90`).
- Stop notice text "before the runs held here go on" with no runs held: with no entries there is nothing to hold, and the stop's way forward is still named.
- Retry window against PATTERN-daemon-wakeups: the wait is bounded at ten minutes, starts at one second and backs off to thirty, and uses no sub-second literal.
- Comment-convention and PATTERN compliance: no entry is violated; `lymdirs` is the only declarer of the hibernate directory, and the declared PATTERN changes list is empty.
