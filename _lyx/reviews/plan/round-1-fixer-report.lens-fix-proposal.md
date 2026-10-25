```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged each proposal the fixer carried out as proposed: F1, F2's probe, F4, F5, F6, F7, F8, F10, F11 and F12.

**MEDIUM — card 5, `StartCapture`'s errors (F10): "each ends with the way forward: read the named child log, then run `lyx drift capture` again, or `lyx drift defer` followed by `lyx loom resume`."**
The second way forward cannot be carried out when the caller is the `Drift-Baseline` row.
- Card 9 returns any `StartCapture` error other than `ErrBreakawayRefused` "as an error naming the child's log, a resumable stop".
- That error carries no `CaptureHaltPrefix` and is no `blocked` Stuck.
- Card 7's `Defer` "refuses unless the status is `blocked` at `Drift-Await`, or `blocked` at `Drift-Baseline` with an error opening with `CaptureHaltPrefix`".

So at the row's error stop, `lyx drift defer` refuses.
The proposal words one text for both callers, `lyx drift capture` and the row.
It should name `lyx loom resume` for the row case, or the row should map the error to a prefixed Stuck.

There is a smaller case in the same text.
When the record-wait bound expires while the slow child still holds the lock, "run `lyx drift capture` again" meets `StartCapture`'s lock-held refusal first.

**LOW — card 7, `capture-child` (F1): it "releases the job only as its last act, after `RunCapture` returns".**
`RunCapture` releases the capture lock before it returns.
- Card 10's `Drift-Await` commits all of `DirRel()`, the capture-child log included, once the lock probe reads free.
- Any line the child writes to its log between the lock release and the job release lands after that commit.
- In the weft that leaves the log tracked-dirty, which is the F6 hazard again.
- Releasing the job ends the child itself, since the child is in its own kill-on-close job (`internal/proc/namedjob_windows.go`, `holdNamedJobPlatform` assigns the calling process).

So the gap is only what the child logs between `RunCapture`'s return and the release.
The card should say the child logs nothing after `RunCapture` returns, or that it logs the release before the lock is freed.

**LOW — card 14 Tests (F4): "`TestRouting_ProgressAt`'s tail subtest expects `Code-Bouncer` `OnDone` `Drift-Verify` …".**
The subtest's name lists the tail's rows: "the tail after Webster runs Merge-In, the Code-Review perch, Describe, PR-Review and Finalize" (`internal/loomrecipe/progress_test.go:55`).
The fix adds `Drift-Verify` to the tail without renaming the subtest, so its title goes stale.

The other proposals hold:
- **F2's probe:** a seat probe matches a live chair by its spec, whose `OutputFiles` stay the same whichever compare log the table is told (`seatSpec`, `internal/seatengine/prompt.go:163`, and `TestAttach_OutputFileMatching_ResolvedAbsoluteSet`). So building the table with "the newest compare log" still finds the chair.
- **F4's "every row maps to a step" subtest:** it derives the main line from routing (`shedengine.ProgressAt`), so the Drift rows map with no other edit.
- **F12:** nothing ties the role audience to a command a stencil names; an index note annotation exists if one were wanted.
- **F5, F6, F7, F8 and F11:** I found no case where the proposal is wrong.

Tool calls:
1. `progress_test.go` routing assertions: my context lacked which subtests pin routing.
2. `seatSpec` and the attach probe: my context lacked what a live-chair probe matches on.
3. `AudienceRole` users: my context lacked whether a guard ties stencil-named commands to the role audience.
4. The opening of `TestRouting_ProgressAt`: my context lacked its "every row maps to a step" subtest body.
5. `shedengine.ProgressAt`: my context lacked how the step mapping is derived.
