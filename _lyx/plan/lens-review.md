# Lens review

Five lens forks reviewed the plan: coverage, row-validity, forced-edits, guard-reach and way-forward; each returned a report.
One line per finding; duplicates across lenses name the line they merge into.

## forced-edits

- FE1 forced-edits BLOCKING — `envkit.FullEnv` keeps the `Webster-Review` segment key after card 22's rename: fixed as proposed — card 22 lists `internal/testkit/envkit#FullEnv` and keys it `Code-Review`.
- FE2 forced-edits MEDIUM — batten's `awaitingHandOff` names the removed verbs: fixed as proposed — card 43 lists `internal/battenshed#awaitingHandOff` and the five awaiting tests that assert it, and drops the PR-Gate clause.
- FE3 forced-edits MEDIUM — `implementer-spec.md` pins Webster's old `on_done`: fixed as proposed — card 22 lists the spec and names `Merge-In` as Webster's `on_done`.
- FE4 forced-edits MEDIUM — the loom describe stencil carries only Plan-Review and Webster-Review carry-over entries: fixed as proposed — card 32 names the Code-Review entry in that rule.
- FE5 forced-edits MEDIUM — `tools/tokencount` reads only the `webster` review directory: fixed as proposed — card 50 lists `tools/tokencount/profile.go` and `main.go` and reads `code`, and `webster` for old runs.
- FE6 forced-edits LOW — `pattern/PATTERN-test-speed.md` names the `publish` gate and the round gate's failure-record step: fixed as proposed — card 52 lists the file.
- FE7 forced-edits LOW — `TestRows_PatternsDenyAndAllow`'s stated reason names the deleted publish-failure steps: fixed as proposed — card 15 lists `internal/shellguard#TestRows_PatternsDenyAndAllow` and rewords the reason.
- FE8 forced-edits LOW — `internal/failuretrace/doc.go` names the Publish failure record: fixed as proposed — card 46 lists the doc.
- FE9 forced-edits LOW — `internal/orchcli/doc.go`, `internal/loomcli/circling.go`, `internal/websterengine/doc.go` and `pattern/PATTERN-plan-generation.md` name removed verbs or Webster-Review: fixed as proposed — card 52 lists all four.

## row-validity

- RV1 row-validity MEDIUM — card 1's parked-state assertion cannot be reached over a clean merge: fixed as proposed — card 1 adds a conflicting row that parks and asserts the source, beside the clean row.
- RV2 row-validity MEDIUM — card 8's expected line count is undefined: fixed as proposed — card 8 runs `--numstat -M`, uses a pure rename, and states the count as six (five text lines plus the binary's one).
- RV3 row-validity MEDIUM — the shared fake seat runner is undeclared: fixed as proposed — card 11 creates `fakeFixerSeats` and its `Run`, and cards 26, 29 and 30 list it under `Uses`.
- RV4 row-validity MEDIUM — card 26's baseline row cannot fail: fixed as proposed — the fixture's `LineChange` fake records `from`, the rows assert it is the post-merge, post-conflict HEAD, and the fixture gains advancing `TaskHead` and keyed `HoldsWork` fakes.
- RV5 row-validity MEDIUM — card 30 names no hook to release the lock mid-call: fixed as proposed — the test runs over a real `parentlock.Store` and `fakeFixerSeats`, whose turn callback releases the record.
- RV6 row-validity MEDIUM — no test reaches the overlap-note write: fixed as proposed — card 31 edits `TestFinalize_MergeJudgement` with darn large-merge rows that read the note, with and without a merge record (merges CV11).
- RV7 row-validity MEDIUM — no test covers a status-built stop: fixed as proposed — card 35 adds `TestGoDriver_Settle` rows through `goDriver.recheck`.
- RV8 row-validity MEDIUM — card 18 claims the notice criterion without a row: fixed as proposed — card 18 adds `TestInnerRun_HaltedStatesWaitExemptWithoutSpawning` rows for blocked Merge-In and Finalize children: one notice, no goto (merges CV7).
- RV9 row-validity LOW — card 2's empty-tip row repeats `TestResolve_CleanMergeNoSession`: fixed as proposed — the row is dropped.
- RV10 row-validity LOW — card 14's new row is vacuous: fixed as proposed — the row is dropped, and the existing settled-refusal row pins the refusal (merges GR6).
- RV11 row-validity LOW — card 25's no-fast-forward assertion cannot fail: fixed as proposed — dropped.
- RV12 row-validity LOW — card 27's release rows repeat `TestGuardedRows_LockLifecycle`: fixed as proposed — that test alone asserts the release.
- RV13 row-validity LOW — card 28's `taskMerger` fake and the `ConflictReroutes` zero value are unstated: fixed as proposed — card 28 creates `fakeTaskMerger` and states that false takes the darn path.
- RV14 row-validity LOW — card 29's `RerouteReviewed` zero value is unstated: fixed as proposed — a nil seam reads as no review, so a struct-literal Finalize halts.
- RV15 row-validity LOW — card 30's landing row repeats the edited message and order tests: fixed as proposed — those tests carry the trailer and no-GitHub assertions.
- RV16 row-validity LOW — `TestStatusCmd_HandBack` belongs in `TestStatusCmd_FoundEnvelope_GenericCore`: fixed as proposed — card 34 adds rows there and drops the new test.
- RV17 row-validity LOW — `TestInnerRun_ParentLockKeptAtFinalize` belongs in `TestInnerRun_ParentLockReleasesAStalledLoopsRecord`: fixed as proposed — card 40 adds rows there.
- RV18 row-validity LOW — `TestLoomRerouteCapWayForward` re-drives a mechanism shed already pins: disputed, every fix worse — D11 states "A test reaches that clause", and the acceptance criterion pins a fourth Finalize reroute halting with loom's clause. `internal/shedengine`'s `TestRun_RerouteCapHaltsOnFourthRerouteAndResumeGrantsThreeMore` tells its own clause, not `loomRerouteCapWayForward`, and asserting the constant alone reaches no halt.
- RV19 row-validity LOW — card 49 names no log-capture helper: fixed as proposed — card 49 lists `internal/testkit/logcapture#Capture` under `Uses`.

## way-forward

- WF1 way-forward BLOCKING — the `pushed` end fires on lyx's own records push across resumes, so a resume cannot pass it: fixed as proposed (the finding proposed no concrete change) — card 11 reads the task branch's remote-tracking tip with no fetch before and after the seat run, and only a tip the session moved is the `pushed` end. `TaskPushedSince` becomes `TaskRemoteTip` on cards 11 and 12 (merges GR1).
- WF2 way-forward MEDIUM — a timed-out turn is never counted, so a resume grants fresh turns: fixed as proposed — card 11 charges a turn when it starts and writes the record before the run and before each re-prompt (merged with CV3; see that line for the turn bound).
- WF3 way-forward MEDIUM — a resumed Finalize after a `finalize`-mode `escalate` may land without reading the decision: disputed, premise false — Finalize spawns its fixer only on a failing `landing` gate (D10 step 4; card 29 `gateAndFix`). A resume over a gate that still fails spawns the fixer, which reads the amended decision record (card 7's stencil names it). A passing gate leaves nothing the fixer was asked to fix, and D10 names exactly `lyx loom decision add` then `lyx loom resume` as this halt's way forward.
- WF4 way-forward MEDIUM — the lock re-check halt is not handed back, so automatic resumes can land under an owner pause: fixed as proposed — card 30's re-check halts, released and unreadable, open with `LandingStopPrefix`, and card 34's `TestHandsBack` pins the re-check halt.
- WF5 way-forward MEDIUM — an undecodable attempt record ends every resume on an error with no way forward: fixed as proposed — cards 10, 26 and 29 make it a Stuck with Halt naming the file, with the way forward to remove it by its path, then `lyx loom resume`, each with a row.
- WF6 way-forward MEDIUM — `taskMergeBase` errors on an unreadable merge record with no way forward: fixed differently — the proposal was a way forward on the error; card 32 instead reads an unreadable record as absent with a Warn, as Merge-In already treats one, so no reader stops and the next Merge-In or Finalize rewrites it. A row pins the fallback.
- WF7 way-forward LOW — the settled refusal's writer clause names Merge-In for Code-Review, which supposedly cannot change the diff: disputed, premise false — Merge-In's merge fixer commits on the task branch (card 26), and a goto to Merge-In merges a moved parent, so re-running Merge-In can change the Code-Perch's subject, as the refusal's "a changed subject then starts a new generation" states.
- WF8 way-forward LOW — `prReviewEntry`'s refusals give no way forward: fixed as proposed — card 20 gives each refusal its way forward and pins the missing-key row.

## coverage

- CV1 coverage MEDIUM — an absent aligned tip reads as a fresh merge base, so a resumed first Merge-In skips the fixer after `escalate`: fixed differently — the proposal was to write the fork point as the record's aligned tip. That would make the aligned tip non-absent after an `escalate`, where D4 says "the aligned tip stays". Instead cards 10 and 26 fix the fork point once per entry in the attempt record's new `AlignFrom`, which every resume reads, and card 26 adds the escalate-then-resume row whose merge base has moved (merges GR2).
- CV2 coverage MEDIUM — Finalize's pre-fix HEAD is unstated and not persisted: fixed as proposed — card 29 writes the entry's `Baseline` at the first spawn, measures from it, and adds the two-turn row.
- CV3 coverage MEDIUM — a single turn can run past `fixer_timeout_min`: fixed differently — the proposal was a per-turn deadline. The seat engine bounds a session run, not a turn, so card 11's gate closure ends a returning turn that ran past `fixer_timeout_min` as the timeout end, and the session bound (`fixer_timeout_min` × turns left) ends a hung turn. A hard kill at the minute mark would need a seat-engine change no decision licenses. The overview's "fixer turn bound" decision states this, and card 11 pins both rows.
- CV4 coverage MEDIUM — no row drives Merge-In through a real resolver: fixed as proposed — card 25 adds a real-hub row to `TestMergeIn_ConflictSessionLoadsLanguageSkills`: a local parent ahead of its upstream merges the upstream SHA through `Fabric.MergeInAt`.
- CV5 coverage MEDIUM — card 25 has no no-upstream and missing-upstream rows: fixed as proposed — both rows added to `TestMergeIn_MergeTip`.
- CV6 coverage MEDIUM — card 26 has no rows for a spent entry resumed over a failing gate or for the over-cap resume: fixed as proposed — the three rows added.
- CV7 coverage MEDIUM — the batten notice criterion has no row: fixed as proposed (see RV8).
- CV8 coverage MEDIUM — nothing pins that the fix rows run the `landing` call, or the darn tree-scoped Finalize: fixed as proposed — card 33 adds `TestWire_DarnRecipe` and `TestWire_DefaultConfig` assertions, and card 29 a darn tree-scoped row.
- CV9 coverage MEDIUM — no row asserts that `approve` and `reject` are unknown commands: fixed as proposed — card 43 adds `TestVerbRefusals` rows.
- CV10 coverage MEDIUM — no end-to-end reconcile row and no `profiles.yaml`-unchanged assertion: fixed as proposed — card 48 adds the row to `TestConfigOverRealHub`, which also seeds from an old profile.
- CV11 coverage MEDIUM — the overlap note's content is untested: fixed as proposed (see RV6).
- CV12 coverage LOW — the LLM driver criterion is prose-only: disputed, every fix worse — the plan spec says a stencil's static prose is not behavior a test covers, and the Plan-Review rubric's test-economy check flags "a card that plans an assertion on a stencil's static prose". The mark the prose branches on is pinned in Go by cards 34, 35 and 37.
- CV13 coverage LOW — card 1's unresolvable-SHA branch has no row: fixed as proposed — card 1 adds the row (merges GR7).
- CV14 coverage LOW — card 9's kind-cap rows are incomplete: fixed as proposed — one row per gate or build configuration name and per added skip or constraint line.
- CV15 coverage LOW — unreached branches on cards 24, 32, 28, 31, 40, 45 and 20: fixed as proposed — each card adds its row: an undecodable earlier record, the merge-record fallback, an unreadable record at Finalize, a `reject_review` generation record, a failing `KeepsLock`, an old pr-gate attempt and a missing `pr_review_setting` key.
- CV16 coverage LOW — no row covers removing the record on a skipped gate: fixed as proposed — card 46 adds a `TestVerifyGate_CheckMapsVerifyResult` row.
- CV17 coverage LOW — a failed spawn's transient class is unstated: fixed as proposed — card 11 marks the start error transient through `internal/shedengine`'s mark (`PATTERN-transient-stop`) and pins it.
- CV18 coverage LOW — the fate of `shedrun.SettleVerifyRetry` is unstated: fixed differently — the proposal left the choice between keeping and deleting; card 18 states it stays so older settle records decode (`PATTERN-tolerant-records`), with no writer or reader acting on it.
- CV19 coverage LOW — card 32 says the loom comment lint takes `VerifyMergeBase`: fixed as proposed — card 32 names the headroom lint in loom's round form and both lints in darn's.
- CV20 coverage LOW — the PR-Rework figure's removal has no row: fixed as proposed — card 50's timing row pins old PR-Rework rows still phased and no PR-Rework span for a new-recipe run.

## guard-reach

- GR1 guard-reach BLOCKING — the `pushed` guard fires on lyx's own push: fixed (see WF1).
- GR2 guard-reach BLOCKING — a resumed first Merge-In drops a pending escalation: fixed (see CV1).
- GR3 guard-reach MEDIUM — Finalize's parent-pair refusals are reached only from the behind state: fixed as proposed — card 27's `syncParent` probes the pair (missing, dirty, mid-merge) on every path before any merge, `parentMerger` gains `CodeDirtyPaths`, and rows pin each refusal for equal, no-upstream and push-skipped parents.
- GR4 guard-reach MEDIUM — the lock re-check does not run again before the retry's landing: fixed as proposed — card 30 re-checks before every landing attempt, the retry's included, with a row.
- GR5 guard-reach MEDIUM — with no usable base the verify gate runs the told `final` call: fixed as proposed — card 33's no-base fallback renders the told list's `--all` call, with a `TestVerifyGate_RoundScenario` row over differing `landing` and `final` lists.
- GR6 guard-reach LOW — card 14's new row has no input that reaches the guard: fixed (see RV10).
- GR7 guard-reach LOW — no row reaches card 1's unresolvable-SHA refusal: fixed (see CV13).
