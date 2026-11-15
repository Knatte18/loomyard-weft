```yaml
lens: fix-affected-set
outcome: reported
```

Checked the fix's affected sets for F3, F8 and F10, rerunning each finding's search. F1, F2, F6, F14 and F15 have no set premise beyond what the fix names, so no rerun was needed.

Unchecked: none of the set-premise findings. F12's "inputs reaching the branch" was not rerun because it is a single input, not a set.

**LOW — card 11, F8 fix: "`strandedEnded` keeps that clause itself … `RunView.ReachedParent` and `DobbyRuns`' reached-parent read of a paused run stay for it."**
The fix keeps the clause in `strandedEnded`, but its doc still gives the reason as dobby giving no pass, and that no longer holds.
- `internal/dobbyengine/ended.go:31-32` says "dobby gives it no pass: batten's state is neither running, nor paused with its work on its parent, nor done", and `internal/dobbyengine/doc.go:285` says the same.
- After the fix, a paused run whose work reached its parent also gets no pass (the act rule waits it). It is excluded from `strandedEnded` for a different reason: its Land is unfinished.
- Card 11 edits `strandedEnded`'s comment only for the cancel marker, and doc.go only for its pass-planning paragraph and cancel-marker paragraphs. Neither bullet rewords these two sentences.
- The `drivable` doc at `internal/dobbyengine/passes.go:69-70` ("running, or paused with its work already on its parent … only its landing and teardown finish it") goes stale too. Card 11 lists `drivable` under Edit but states no doc change.

**LOW — cards 8 and 11, F10 fix: "every other paragraph that names the cancel marker names the unfinished cancel intent".**
That sentence covers doc.go only. Rerunning the marker-prose search finds survivors in functions no card's comment bullet names.
- `internal/battencli/cancel.go:405-413` (`isBeamProject`'s doc: "before the cancel marker is taken") is not on any card's Edit list.
- `internal/dobbyengine/passes.go:111`, `planPass`'s doc ("none while the cancel marker exists"): `planPass` is under Edit in card 11, but card 11's comment bullet names only `strandedEnded`, `readWaits` and `runNotices`.
- `internal/battencli/cancel.go:487`, `clearRunPhase`'s doc ("The cancel marker and the pause gate stay with the caller"): `clearRunPhase` is listed in card 7, but card 7 states no change to that comment.
- `internal/pairteardown/teardown.go:713` ("running with no cancel marker") sits beside `registeredBattenAlive`, which card 8 edits, but card 8's comment bullet names only `pairteardown/doc.go`.

**F3 rerun: no finding.** Every test that asserts the written status (`commitstatus_test.go:134`, `wiring_commitstatus_test.go:127` in `TestNewCommitStatusSeam_BoardStatus`, `wire_seams_test.go:697/714`, `lifecycle_integration_test.go:1067`) is now on card 6. The other `running · …` hits are seeds or shape tests. `loomcli/peers.go` reads only the producer half.

**F8 rerun: no finding beyond the one above.** The `ReachedParent` users are `passes_test.go` (TestPlanPass and TestDrivePasses_ExitHandling, both listed), `ended_test.go:28` (stays false, which is right), `dobbypairanchor_test.go` (unchanged, still valid) and `dobbyreads.go:203-216` (kept).

Tool calls:
- Read `adopt_integration_test.go:65-80` and grepped `running · `/RunStatus across internal, contracts and docs: the context lacked the full set of status assertions for F3.
- Grepped the loom commit-status test names and the halted/paused status strings: needed to place `wiring_commitstatus_test.go:127` in its test.
- Grepped `peers.go` for how it reads the status, and listed files naming the cancel marker: F3's reader set and F10's prose set were not in context.
- Grepped the cancel-marker lines in the candidate files: needed the exact surviving sites for F10.
- Listed the function starts in `cancel.go` lines 230-490: needed to attribute the surviving cancel.go comments to their functions.
