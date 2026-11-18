---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: The proposed change returned an error at the first unreadable or undecodable registry entry, before comparing its session id; the registry holds every Claude session's entries, so one stray bad file would disable the holder lookup for every session and for the orch session-start registration (internal/orchcli/start.go:203). sessionHolder instead keeps scanning, returns a live holder found in any entry, and returns the remembered entry error only when no live holder is found, so the step stop still reads such a run as held.
---

## Changes

- F1, `internal/shedverbs/loopwait.go` `loopExitStop`: for a gone loop with a child in flight under a trace id, it keeps an interrupted record at `<StepsDir>/<TraceID>.json` before the envelope is delivered, unless a record exists.
  The record carries the stop's message and way forward, the child's first trace file or the loop log, and the friction note.
  `newestInterruptedTraceID` then finds the dead loop's in-flight child on the re-step.
  The teardown mark, a loop that never recorded itself, and a busy stop write none.
  The change sits in `loopExitStop` and not `deadLoopStop` because the friction note and trace file it needs are computed there; `deadLoopStop` is its only caller that records a child.
  Test: a `TestUntilStop_DeadLoopStopsLoopExited` case runs the dead-loop stop, asserts the newest interrupted trace id, and asserts `reconcileInterruptedStep` hands that id to `OnStepOrphaned` on the failed status.
- F2, `internal/shuttleengine/claudeengine/resume.go` `sessionHolder`: an unreadable or undecodable registry entry is now an error when no live holder is found, naming the entry; a live holder in another entry is still returned.
  `sessionMayBeHeld` reads the error as held, so the run is listed under Held and keeps running.
  Tests: `TestSessionHolder` rows for an undecodable entry alone and for a live holder behind an undecodable entry; `TestRunner_StopStepRuns_StopsOnlyTheStepsRuns` gains a failing-lookup run asserted as Held and still running.
  The `ReadErr` branch has no test row, since a portable unreadable registry file cannot be built in a test.

## Self-check

- fix-code-correctness, LOW: the stop-test lookup-failed row pins `sessionMayBeHeld`'s existing error-reads-held branch and passes without the `resume.go` change — not a defect; the `claudeengine` rows cover F2's defect. Disposition: fixed as proposed (no change needed).
- fix-code-correctness, LOW: `sessionHolder` errors for a bad entry of another session — the review's proposal; narrowed by the fix-proposal finding below. Disposition: fixed differently, with the scan-on and error-only-without-a-holder form.
- fix-consistency, MEDIUM: `sessionHolder` aborted at the first bad entry and lost a holder `StartSession` used to find. Disposition: fixed (scan continues, a live holder is returned).
- fix-proposal, MEDIUM: one bad registry file disables the lookup for every session. Merged with the fix-consistency finding. Disposition: fixed (same change).
- fix-proposal, LOW: the same error reaches `StartSession` and `relaunchForReprompt`. Disposition: fixed by the same change for `StartSession`; `relaunchForReprompt` still refuses when no holder is found and an entry is unreadable, which is the intended guard.
- fix-proposal, LOW: the dead-loop record's `error` omits the stop's detail. Disposition: not changed; `newestInterruptedTraceID` reads only `kind` and `trace_id`, and the stop's detail is in the delivered envelope and the status file.
- fix-affected-set: no findings; it listed the missing `ReadErr` row as unchecked, which is named under Changes as not testable portably.

- Gate repair: two doc-comment lines the round added reached 120 characters, in `loopExitStop`'s doc and `sessionHolder`'s doc.
  Each is now split at a sentence boundary; no behaviour changed.

## Disputed

None.

## Deferred

None.

## Rulings

None.
