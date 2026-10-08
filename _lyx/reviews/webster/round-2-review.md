---
verdict: APPROVED
findings:
  - id: F1
    severity: NIT
    class: consistency
    location: PATTERN.md:67
    summary: the PATTERN-pane-binary-resolution index entry and its background opening sentence still scope the rule to strand panes, while the bullets and the enforcement test now cover the detached lyx window too
---

Range: `product.parent` is `main` and `_lyx/loom/rework/` does not exist, so the range is `git diff $(git merge-base main HEAD)..HEAD`.
Evidence run: `go build ./...`, `go vet` under the default, `integration`, `tmux` and `llm` tags, `go test ./...`, `go test -tags integration` over the touched packages, and `lyx loom lint-comments --base <merge-base>` all pass.

### [NIT] The pane-binary invariant's index entry still names strand panes only

**Location:** `PATTERN.md:67`, `pattern/PATTERN-pane-binary-resolution.md:3`
**Issue:** The diff extends `PATTERN-pane-binary-resolution` to the detached `lyx` window.
`panebin.go` now has two call sites, `launchStrandLocked` and `OpenWindow`.
`panebin_enforcement_test.go` now fails on a `new-window` literal outside its allowlist.
The background file's bullets were updated.
Its opening sentence ("A strand pane reed creates resolves `lyx` to the binary that spawned it") and the `PATTERN.md` index line ("Creating a strand pane in reed: ... through the one chokepoint in `panebin.go`") still state the applicability as strand panes only.
`PATTERN.md` is the authoritative index, so a reader who creates a window and checks the index finds no rule that applies to them.
**Fix:** Widen the applicability clause of the index line and the background's opening sentence to cover the detached `lyx` window `OpenWindow` opens.

## Focus verdicts

1. **Per-check process cost in the wait.** Not a defect.
   `ReadDecision`, `StopReport` and `AttachDir` all resolve the task worktree through `taskWorktreeLocator`, which spawns git (`lyxcwd.ResolveWorktree`) until the first success and afterwards only stats the pair, so no check after the first spawns a process.
   `ReadDecision` is two file reads, `StopReport` a stat and a read.
   `awaitingStep`, `noticeStep` and `stateChangeReady` call the seams every check, but `DriverAlive`, `DriverStrand`, `ReviewWait`, `Activity`, `MarkWatched` and the not-parked resume retry sit behind `w.probeDue`, the cached `aliveStale`/`activityStale` readings, or the `noticeProbe` spacing in `deliver`.
   `ResolveParent` and the orch-state read run only inside `MarkWatched`, which runs on probe checks.
   No test counts resolutions over a long wait.
   `stepTaskWorktreeSeams_RunGitOnlyUntilTheFirstResolution` instead hides the pair's `.git` after the first resolution and shows `AttachDir`, `ReadDecision` and `StopReport` keep answering.
   `TestInnerRun_RecordsWhetherItHoldsTheWatchedMarker`, `TestInnerRun_RunningArmReviewWaitNote` and `TestInnerRun_DoneFinishesPastGraceWithLiveDriverAndWarns` bound the probe-gated seams.
   Together these pin the property, so I raise no coverage finding.
   `internal/battenshed/doc.go` matches the code on which work runs per check and which per probe.
2. **`Down` over a corrupt `reed.json`.** Not a defect.
   `sessionReapRootsLocked` lists `list-panes -s` through `reapSessionPanes`, which reads no reed state and covers every window, so a batten window's panes are reaped too.
   The plan said the session-kill reap keeps the current-window fallback; listing all windows is a superset and the correct reading, since `kill-session` ends every window.
   Its doc comment says it reads no reed state.
   The untouched `Down` doc comment says it "loads no state" in the foreign-session context, while `Down` reads state afterwards only for the best-effort abandonment report; I leave it because the diff did not change that text.
   `TestDown_ListsEveryWindowsPanesOverACorruptState` writes a corrupt state, drives `Down` and asserts the session-wide listing precedes `kill-session` and the state file is gone; the old per-window listing would fail it.
3. **Running-arm reason and the watched stencil.** Not a defect.
   The reason "inner shed run still running; checking it every `<poll>`" matches a row that stats the status file each poll interval inside its call, and `TestInnerRun_RunningArmReviewWaitNote` pins it.
   `shed-template-driver-notify-watched.md` states the driver's own three cases and a "send nothing else" rule; its closing clause "those stops reach the parent by other means" names no announcer, and card 3's own intent uses the same rationale, so I read it as satisfying "says nothing of who announces a stop".
   `claims_test.go` pins the relay, question and `friction: failed` messages and forbids the generic escalation line.
4. **Touched comments.** Compliant apart from F1.
   `lint-comments` against the merge base reports nothing.
   Each touched sentence in `panebin.go`, `doc.go` and the pattern file sits on its own line.
   The `Their EFFECTIVE values are read` break in `doc.go` is carried over from existing wrapped text, which `PATTERN-comment-line-breaks` excludes.
   The `panebin.go` header and the pattern file's first bullet name both `launchStrandLocked` and `OpenWindow`, which calls `composeWindowCommand` in `window.go`; `doc.go`'s Detached windows paragraph says the command is composed in `panebin.go`.

## Other checks against the plan

Cards 1 to 11 each match their intent in the code I read.
Two deliberate deviations are documented in code and correct: `paneGenerationLocked` keeps the `=<session>:` target because a bare `=<session>` expands session fields to empty (stated in `generation.go` and `doc.go`), and the session reap lists all windows as above.
Card 1's deleted `TestCompact_BoundaryAtEntryCompletes` has its case folded into `TestCompact_CompletesOnBoundaryAfterEntryAndIdle`.
Test economy: new test functions map to cases the plan names; I found no duplicate whose covering test I could name.
No Rename or Delete groups appear in the cards, so the per-type mechanical checks have nothing to run.
