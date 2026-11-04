```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff. Each item gives the case I ran and whether it came out correct.

**Per-function list**

1. `rememberRuns`, run ahead of Land, then past Land with the request set: one line, then a clear. Correct.
2. `rememberRuns`, request raised in the tick the run lands (`requests` true, `note.Requested` false): the line is decided once and `takeDoneEntries` skips the remembered slug. Correct.
3. `rememberRuns`, per-task with no request: the memory is dropped. Correct.
4. `rememberRuns`, `off` with a request: `ClearOwed` is kept and no line is made. Correct.
5. `rememberRuns`, `modeRead` false: nothing is decided, and `takeDoneEntries` does not run. Correct.
6. `takeDoneEntries`, done and flagged entry, no memory, run unlisted: line `landed, commit unknown` plus a clear. Correct against the summary. The caveat is item F2.
7. `takeDoneEntries`, run listed past Land: the line names the commit. Correct.
8. `takeDoneEntries`, other parent worktree: skipped. Correct.
9. `settleLandNotices`, post fails: the line stays owed, the clear is not attempted, and there is no repost. Correct.
10. `settleLandNotices`, post succeeds and the clear fails: `Line` is reset first, so only the clear is retried next tick. Correct. The one hole is a failed `updateLandNotice(Line="")`, which causes a duplicate line, i.e. at-least-once delivery.
11. `settleLandNotices`, clear succeeds: the memory is deleted. Correct.
12. `settleLandNotices`, `ClearNotifyOnLand` nil: wrong. See F3.
13. State record lost: the entry stays flagged and done, so it is retaken. Correct.
14. `DobbyTasks`, flagged done entry whose `parents.Of` fails: wrong. See F4.
15. `Store.Prune`: a project is kept while a surviving flagged entry names it, and `depends_on` and `order` are stripped only for removed slugs. Correct.
16. `refuseNoticeOnAbsentEntry`: a payload of exactly slug plus `notify_on_land` true on an absent slug is refused. A payload with `id`, a non-string slug, a found entry or `false` passes through. Correct.
17. `landedNotice`: wrong. See F5.
18. `isHubOrchName` and `checkReceiverLocked`: `ly:gone:orch` is accepted; `zz:orch` (another hub) and `ly:gone:orch-2` (a thread) are refused. The rule is correct, but the stored mail does not survive. See F1.
19. Call paths for `checkReceiverLocked`:
    - The Forward path (`verbs.go:840`) now also accepts an unlisted hub orch. Harmless, since it uses the same mail path.
    - A mistyped `<shortname>:<slug>:orch` is accepted and stored. This ends in the F1 outcome.

**Findings**

**F1. HIGH, design. Mail to an unlisted orch is returned undelivered by creel's sweep, not kept until the orch is back.**
- `checkReceiverLocked` now stores the post. The mail is journaled `posted` with `Holder` equal to the unlisted orch name.
- `sweepSubjects` counts it, because `isStrandName` is true for an orch name and the message is unread (`sweep.go:184`).
- `sweepOnce` finds no strand of that name: the first miss at 30 s, the confirming sweep at 10 s, `goneAfterSweeps = 2` (`sweep.go:22,140-146`).
- `releaseGoneHolder` then sends each unread message back along its path (`sweep.go:218-228`). The sender is dobby's program name, which is not a strand name. The message is journaled `eventUndelivered` and any wait on it is answered `returned`.
- The notice is therefore dropped about 40 s after the post, and dobby has already marked it sent, cleared its request and dropped its memory. Goal item 3 and the doc and summary claim ("stays unread in the pool and is delivered once the orch's strand is back") are not met beyond that window. It is worse than before, because the loss is silent.
- The new creel test (`verbs_test.go` row `Post_ToAnOrchOfTheHubWithNoStrandWaitsForItsStrand`) never starts the scheduler or runs a sweep, so it cannot see this.
- Existing proof of the sweep behaviour: `flags_test.go:104-110` (`ReturnMovesTheFlagBackToTheSender`) and `TestSweep_RemovedHolders`.
- Evidence: `internal/creelengine/sweep.go:134-148,184,206-229`; `internal/creelengine/pool.go` (`isHubOrchName`); `internal/creelengine/verbs.go:62-75,168-193`.

**F2. LOW, design. A hand-marked done entry gets a "landed" notice.**
- `takeDoneEntries` builds a default `RunView{PastLand: true, Landed: true}` for any done, flagged entry of this worktree with no listed run.
- `boardengine.StatusDone` is documented as also meaning "marked done by hand". Such an entry would produce "landed, commit unknown" for a run that never landed.
- A done flagged project (`kind: project`) gets the same line.
- Evidence: `internal/dobbyengine/landnotice.go:102-120`; `internal/boardengine/store.go` (`StatusDone` comment).

**F3. MEDIUM, design. With `Deps.ClearNotifyOnLand` nil, a requested done entry is re-noticed every tick.**
- `settleLandNotices` posts the line, skips the clear (`d.deps.ClearNotifyOnLand != nil` is false) and deletes the memory.
- The entry stays done and flagged, with no memory, so `takeDoneEntries` retakes it next tick and the line repeats forever.
- The path is `beamcli` wiring today, which always sets the field, so it is reachable only through a nil seam. The `Deps` doc says "nil clears nothing", but the new design now depends on the clear to terminate.
- Evidence: `internal/dobbyengine/landnotice.go:171-177,102-118`; `internal/dobbyengine/loop.go:128-129`; `internal/beamcli/dobbydeps.go:171`.

**F4. MEDIUM, design. One unreadable flagged entry aborts all of `DobbyTasks`.**
- `DobbyTasks` now calls `parents.Of` for every done entry carrying `notify_on_land`. An error from `hubgeom.ParentOf` is returned (`read the parent of …`), so the whole task list fails.
- That fails the tick's start step for every slug, and `landNoticeTasks` goes to `Fail`, so no done entry is taken up that tick.
- `dobbyParentRuns` handles the same error per run: Warn and skip, except `ErrInvalidAnchor`. This change makes the failure persistent for as long as the entry is flagged.
- For a landed entry the origin record is the thing most likely to be gone or unreadable. When no record exists, `ParentOf` falls through to `ParentBeam` on the local board and may resolve to the prime, not the real parent.
- Evidence: `internal/battencli/dobbyreads.go:284-296`; `internal/hubgeom/parentof.go:21-32`; `internal/battencli/dobbyreads.go:85-92`.

**F5. LOW, design. `landedNotice` fires on any upsert of a flagged done entry.**
- It tests only `task.NotifyOnLand && IsDoneStatus(task.Status)` on the returned task.
- An upsert that edits only the title or body of an already flagged done entry therefore claims the run "already landed" and that the line is sent next tick.
- The text is also wrong under `land_notice: off`, where dobby sends no line and only clears the field.
- Evidence: `internal/boardcli/cli.go` (`landedNotice`, upsert `joinNotices` call).

**Tool calls (1)**
1. `sed`, `grep` and `sed` over `verbs.go:825-850`, `appendPostedLocked`, `ValidateRole` and `flags_test.go:95-125`. My context lacked the Forward path, how posted mail sets its holder, and the existing sweep-return test that confirms the F1 behaviour.
