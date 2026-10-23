# Fixer report — discussion round 1

Every finding's premise was checked against the code it cites, and each holds.

## Changes

- F1 (BLOCKING): D11 now says batten's binary re-step skips a blocked stop with the reroute-cap or landing-stop prefix, a check beside its blocked-or-failed test, so only `lyx loom resume` lifts these halts; every other halt keeps the re-step. Added the criterion that a changed build does not re-step a Finalize reroute-cap halt, a Merge-In escalation or a fixer-spent halt.
- F2: D11's Added list makes `arbiterstop.Eligibility` refuse the same stops, ending with `HandBackWayForward`. The Go-driver criterion now covers the arbiter verb's refusal.
- F3: D4 states the outcome of every other fixer failure exit as a landing-stop halt, and the Go-driver criterion now covers the timeout. The self-check later took the failed spawn out of the halts (see Self-check).
- F4: D10 step 4 counts a change touching a `_test.go` file, `testdata`, gate or build configuration, or adding a skip or build constraint, as over the cap whatever its line count. D10 states that the size and kind caps bound an unreviewed Finalize fix. Added a criterion for a test-file edit under 40 lines.
- F5, fixed differently: D10 step 4 halts with the landing-stop prefix on a spent cap or over-cap change when the route back reaches no review (darn's budget-0 `PR-Review` edge, or loom at `code_max_bounces: 0`). The rationale now names that exception, and a budget-0 darn criterion was added. I did not make the proposed change to measure the caps from the first Finalize entry since the row's latest Done: under the halt, a reroute happens only when a perch reviews on the way back, and a cumulative measure would then count perch-reviewed fixes made between passes against the Finalize cap.
- F6: D4 says an absent aligned tip reads as the task's fork point, and that Merge-In writes the aligned tip on every Done, the no-spawn path included.
- F7: D10 step 3 takes the task side from the task's merge base to the pre-merge task head, as the Merge-Review skip seam's computation does. D10's overlap test was dropped from D3's reader list.
- F8: D10 step 3 hands Darn-Bouncer the overlapping parent changes through a Bouncer re-entry seam fed from the merge record, and the perch runs an amendment generation even over an unchanged task diff. Added a criterion for a darn overlap-only reroute.
- F9: D10 step 4 states the finalize-mode `escalate` outcome: keep the commits, release the lock, halt with the landing-stop prefix, way forward `lyx loom decision add`, then `lyx loom resume`.
- F10: D10 says the idle backstop never releases a lock while the owner's run is at Finalize and running, or while a verify marker is live. The dead-Finalize sentence now relies on `internal/parentlock` granting a record whose owner process died (`store.go:249`). Added a criterion that a long gate run keeps the lock.
- F11: D13 states what the deploy now runs and what bounds it, and that `internal/landingshed`'s `doc.go` loses its PR-Gate-reviewed claim. "F4's kind cap" is cited as D10's size and kind caps.
- F12: D4's fixer report names each parent change it judged and its finding, and D4 states what the advance skips and that bound. D5 adds the latest `merge-in` report to the Code-Perch's review input.
- F13: D10's bound sentence now covers the retry pass's second run of steps 2 to 4.
- F14: D14 names each module's own migration (loom, landing, a new gate migration), orders the loom table after `retiredReviewKeyRenames` and `copyWebsterModelsOntoMerge`, and has `PrepareRunSettings` log `Migrate`'s lines at Warn. Scope's config bullet and the Constraints cross-reference follow.
- F15: D4 removes the reviewed-tip file, `MergeReviewedPath` and `RecordMergeReviewed`, and keeps the aligned tip only in the merge record.
- F16: D12 says a skipped gate over a tree `verifytree` records as passed removes the record.
- F17: the lock-free Merge-In criterion is limited to a parent with an upstream and no push skip. Added a criterion that the push-skipped read holds the lock only for the read.
- F18: D10 step 1 keeps the parent-pair Stucks under the lock before the merge.
- F19: the first Open risks bullet is cut, and the second reads as the proposal worded it. The 73% / $0.96 figure moved to the support log's Turn 5 alignment entry.
- Follow-ons: D13's orch rule and the `PATTERN-parent-lock` change now name the Finalize landing-stop halts and the no-review halt.

## Self-check

- fix-claims: no findings.
- fix-consistency, LOW, darn large-merge hand-off ignores the budget-0 recipe: fixed. Step 3 is limited to a budget above 0 and says what budget 0 does; the darn reroute criterion names PR-Review.
- fix-consistency, LOW, Scope In omits the new battenshed, arbiterstop, sheddriver, shedadapters, gate and configreg work: fixed with three Scope bullets.
- fix-consistency, LOW, D2's twin still names only the fast-forward: fixed. D2 now says the parent-pair Stucks move into Finalize.
- fix-way-forward, LOW, the new fixer failure halts state no way forward: fixed. D4 names each one's way forward, including the pushed-branch halt.
- fix-proposal, MEDIUM, a re-entered Code-Perch never reads a new fixer report over an unchanged diff: fixed. D5 runs an amendment generation over a fixer report newer than the settled verdict, through the same re-entry seam.
- fix-proposal, MEDIUM, F18's ahead or diverged Stuck lacks the push-skip exemption: fixed. Step 1 applies that Stuck only with no push skip (`mergein.go:216`).
- fix-proposal, LOW, a failed spawn became an orch-only halt: fixed. A failed spawn is a transient error stop the arbiter or re-step may resume, and the row's spent attempts persist in the run's scratch directory so a resume grants no fresh cap.
- fix-proposal, LOW, F8's review input has no base for a darn run with no merge record: fixed. It falls back to the task's merge base.

## Disputed

None.

## Deferred

None.
