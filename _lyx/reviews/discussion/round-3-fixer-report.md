# Fixer report — discussion round 3

## Changes

- F1 (BLOCKING), fixed as proposed.
  D3 now names the stage before the run lock: `battenPreRun` and `battenPreStepUnmarked` in `internal/battencli/arm.go`, which seed `Worktree-Create` through `state.UpdateJSON` after `battenshed.ClearRunMarkers` without holding the run lock.
  The pre-drive check has two call sites: inside those seeding closures, under the status lock, and in shed's `Run` and `Step` behind the run lock for a resume.
  The marker clear moves into the closure behind the check, so a refused seeding writes no status and clears no marker.
  Criterion 263 now says the step refuses in the seeding closure or at the run lock, writing no status, no seed and no claim, and that a hook running after the marker's removal meets the missing seed before any status write.
  The Scope line for `internal/battencli` names the new call site.
- F2 (MEDIUM), fixed differently.
  D4 and the plan-writer note now say the marker covers a tackler the driver had recorded, and D4 names the window between `Shuttle.Start` and `recordStartedTackler`.
  The window is listed under Open risks.
  Not made: the proposed bound, "the tackler's deadline from its settings".
  Shuttle checks the deadline in the waiter's `Wait` (`internal/shuttleengine/run.go`, the `deadline` field of a `Run`), so once the driver is ended nothing enforces it.
  The risk instead bounds the window, which is the two file writes after `Shuttle.Start`, and says the strand lasts until its pane is closed by hand.
  The research branch was also rejected.
  In the prime the tackler's strand is named `<shortname>:tackler` (`internal/hubgeom/hubgeom.go:34` leaves `NameSlug` empty in the prime), so listing strands cannot tell one run's tackler from another's.
  Support-log Q22 records that pick.
- F3 (MEDIUM), fixed as proposed.
  D4 adds a Windows re-list after step 2's lock takes and before step 3's strand stop, using the same creation-time filter.
  It refuses with each pid, role and `taskkill` line and leaves the marker.
  On Linux, `End`'s survivors error does the same work.
  The survivor sentence in D4 and criterion 267 now say the re-list produces the Windows refusal.
- F4 (MEDIUM), fixed as proposed.
  D5's start refusal now covers a left task worktree, weft worktree or local weft branch.
  These are read locally with no network call, and the refusal names `lyx batten cancel <slug> --remove`.
  It explains that Add refuses a weft worktree with a bare error and adopts a left weft branch.
  A task branch left on its own stays with D11's Worktree-Create halt.
  A new criterion covers a plain cancel over a pair whose task worktree was already gone and whose weft side was left, followed by `lyx batten run`.
  The D5 auto-mode assumption and the Scope line are updated to match.
- F5 (LOW), fixed as proposed.
  When no `--remove` was given and a task branch is left locally, D2's nothing-left refusal names `lyx batten cancel <slug> --remove`.
  Criterion 274 now says a plain cancel in that state refuses that way.
- `support-log.md`: a Round 3 section lists these fixes.

## Self-check

- fix-consistency, LOW: the D3 auto-mode assumption and support-log Q18 still placed the gate only behind the run lock. Fixed as proposed: both now name the seeding-closure call site.
- fix-consistency, LOW: criterion 262 omitted the pre-drive check's refusal of an unregistered batten. Fixed as proposed: the criterion adds that route.
- fix-claims, LOW: the claim that the prime-side tackler name carries no slug was unconfirmed. Fixed as proposed: confirmed against `internal/hubgeom/hubgeom.go:34`, which the open risk now cites.
- fix-way-forward, LOW: D2's nothing-left list said only "no pair", so it was unclear whether a lone local weft branch counted as something left. Fixed with own change: the list now names the pair's local non-task parts: task and weft worktrees, local weft branch and launcher entry. The first rewording pointed at D5's left-part list, but that list includes the task branch, which would contradict F5's plain-cancel case.
- fix-way-forward, LOW: the Windows re-list refusal did not name the cancel again as its next step. Fixed as proposed: the refusal now names `lyx batten cancel <slug>` after the `taskkill` lines.
- fix-proposal: no findings.

## Disputed

None.

## Deferred

None.
