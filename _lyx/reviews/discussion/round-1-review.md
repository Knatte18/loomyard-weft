---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md, D7
    summary: D7 retries the contained removal, but the sighted not-empty failure is git worktree remove's own, which unregisters the worktree, so the fallback that holds the contained removal is never reached and the retry never fires
    fix: Rewrite D7 around the failure that occurs. When git worktree remove exits nonzero and the target is no longer a registered linked worktree, git has already deleted the registration and left the directory; state that premise in D7. Place the bounded retry where that state is handled in removeWarpWorktreeDir (internal/fabricengine/remove.go), today the branch that returns the not-a-linked-worktree error. State the new ownership proof that replaces the registered-linked-worktree check for that leftover directory (for example, the target was registered when this Remove began and is the path git was asked to remove), state how the not-empty failure is recognized (git exits nonzero with a delete failure, then the contained removal fails not-empty), and state per OS what error value counts as not-empty. Drop or reword the sentence claiming a re-entered Remove cannot finish this state, since the sighting records a re-run that finished after the lock files were deleted. Update the D7 acceptance criterion so its scenario starts from git worktree remove failing on a refilled directory, not from the contained removal failing first.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md, D1 and the first acceptance criterion
    summary: D1 accepts a tool event after the session start as readiness proof, but the events file records no general tool hook, so that half of the proof names a signal no engine parser produces
    fix: In D1 and in the first acceptance criterion, drop or a tool event and make the proof a session-start signal followed by a turn-start signal. If a tool signal is wanted, add a Decision that records a tool hook in claudeengine's settings and a parser kind for it, with its bound.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md, D1 rationale
    summary: D1's rationale says After D3 every launch through start carries a first turn, but D2 is the launch-line Decision and D3 is the status pane
    fix: In D1's rationale, change After D3 to After D2.
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md, D2
    summary: D2 moves the first load turn off the verified send onto the launch line but states only the maxLaunchPromptBytes bound, not which guard the move bypasses or what bounds a launch-carried turn that never starts
    fix: Add to D2 a Bound line stating that the launch-carried load message skips the verified send's landing check, that its landing is proven instead by a turn end in the events file, and that a turn that never starts or never ends is bounded by the skill-load timeout, measured from the end of awaitStartup, after which its skills are skipped with a warning and the color and prompt pointer follow.
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md, D6 and its acceptance criterion
    summary: D6 resolves each named path before the containment check but not the worktree root it compares against, leaves a write failure partway through a batch unstated, and its acceptance omits the zero-file refusal, mode preservation and the envelope naming each refused file
    fix: In D6, state that the worktree root is resolved the same way as each named path before the containment check, and state what a write failure after some files are written does (for example, the envelope names the files written and the one that failed, and the call exits nonzero). Extend the lyx loom fmt acceptance criterion with a call naming no file refusing, a rewritten file keeping its mode, and a refused call's envelope naming each refused file.
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md, Acceptance criteria
    summary: Scope In item 4, the weft push finding recorded in pushretry.go's doc comment, has no acceptance criterion, while every other In item has one
    fix: Add an acceptance criterion that the doc comment of gitrepo.PushWithRetry in internal/gitrepo/pushretry.go names the remote ref-update race on the weft's main as the cause of a retried remote rejected (failed) refusal.
---

### [BLOCKING] D7's retry sits on a path the sighted failure never reaches

**Location:** `_lyx/discussion/decision-record.md`, D7

**Issue:**
The board entry's sighting reads: "`git worktree remove` failed with \"Directory not empty\"".
The failure is git's, not fabric's contained removal.
I reproduced the git side: a linked worktree whose directory git cannot fully delete makes `git worktree remove --force` exit 255, and `git worktree list` afterwards no longer lists it, with the directory left on disk.
Git deletes the registration even when the work-tree deletion fails.
In `removeWarpWorktreeDir` (`internal/fabricengine/remove.go`), a `GitError` followed by `!isRegisteredLinkedWorktree(l, target)` returns the "not a linked worktree of this repo, so fabric will not delete the directory itself" error.
`isRegisteredLinkedWorktree` queries the live worktree list, so after git's failure it answers false.
The fallback `removePath` holding `removeContainedPath` is reached only while the target is still registered, and its `ownedRegisteredLinkedWorktree` check would refuse an unregistered target anyway.
D7's retry, placed "inside that one removal call ... after the ownership and dirtiness checks have passed", therefore never runs in the race it targets, and its acceptance criterion would pass in a test while the teardown keeps failing.
D7 also says "A re-entered `Remove` cannot finish this state", while the sighting records that re-running `lyx batten run` finished the teardown once the two lock files were deleted.
"Matched by its error value ... on every OS" is not actionable as written: Go has no portable not-empty sentinel, and Windows reports `ERROR_DIR_NOT_EMPTY`, not `syscall.ENOTEMPTY`.

### [MEDIUM] D1's tool-event proof names a signal the events file never holds

**Location:** `_lyx/discussion/decision-record.md`, D1 and the first acceptance criterion

**Issue:**
D1 makes a launch ready on "a session-start signal and, after it, a turn-start signal or a tool event".
`claudeengine/doc.go` lists the recording hooks: Stop, UserPromptSubmit, StopFailure, Notification, SessionEnd, SessionStart, PermissionRequest and, in an interactive run only, the AskUserQuestion record.
No general `PreToolUse` or `PostToolUse` hook is recorded, and `EventKind` has no tool kind beyond `EventAsk`.
The plan writer cannot implement "a tool event" through the engine's parsers, as D1 requires, without a recording change no Decision makes.
The turn-start half suffices: a positional launch's turn start is recorded (see the focus notes below).

### [LOW] D1 cites D3 for the launch-line change

**Location:** `_lyx/discussion/decision-record.md`, D1 rationale

**Issue:**
"After D3 every launch through `start` carries a first turn" refers to the launch-line change, which is D2.
The support log's Turn 5 shows the Decisions were renumbered ("D3 (now D2)"), and this reference was missed.

### [LOW] D2 does not bound the bypass of the verified send

**Location:** `_lyx/discussion/decision-record.md`, D2

**Issue:**
Today `loadSkillTurn` sends the load message through `sendVerified`, which proves the submission landed.
D2 replaces that send with a launch argument, which bypasses the verified send for the first load turn.
The rubric's attack-surface rule asks the Decision to state what the edge lets through and what bounds it.
D2 states only the one-line `maxLaunchPromptBytes` limit.
It does not say that a launch-carried turn that never starts is caught only by the load timeout, nor when that timeout starts, now that no send marks it.

### [LOW] D6 leaves the root resolution, a partial write and three bounds untested

**Location:** `_lyx/discussion/decision-record.md`, D6 and its acceptance criterion

**Issue:**
D6 resolves each named path, symlinks and junctions included, and then checks containment, but it does not say the worktree root is resolved too.
Comparing a resolved file path against an unresolved root misjudges containment whenever the worktree is reached through a link.
"Checks and formats every named file before writing any" covers refusals, not a write that fails after earlier files were written.
The acceptance criterion omits three of D6's stated bounds: at least one file is required, the file mode is kept, and the envelope names each refused file.

### [LOW] Scope In item 4 has no acceptance criterion

**Location:** `_lyx/discussion/decision-record.md`, Acceptance criteria

**Issue:**
Every other In item maps to a Decision and at least one acceptance criterion.
D4's deliverable, the finding in `pushretry.go`'s doc comment, has no criterion, so nothing checks that it landed.

## Focus notes

- D1, D2 and D3 cross-references: the only mismatch is F3.
- Events-first readiness: a skill-less run in this worktree (`.lyx/shuttle/0a5fb4346bc06a4d96db06edb81e176f`, no `promptLine` in its `run.json`) recorded `SessionStart` and then `UserPromptSubmit` carrying the positional pointer.
  The positional-turn half of the first Auto-mode assumption holds, so it is not an unnamed false-premise risk.
  A resume spec goes through a fresh run directory and a fresh events file, and an engine with `RecordsSessionStarts` false keeps the capture path; neither is a defect.
- D2 ordering: load turn end, color, retry turn, prompt pointer is stated, and the timeout and death paths are tied to today's. The open bound is F4.
- D3: `statuspane`'s loop redraws only when the text or size changes, so the history clear is bounded by content changes, and the copy-mode cost is named in Open risks.
  The redraw never prints more lines than rows and never ends on a newline, so history fills only when tmux pushes rows off a shrinking pane and pulls them back on a growing one.
  The stated cause therefore supports the fix; not a defect.
- D6: F5.
- D7: F1. The retry count, wait and matched error are stated in the Decision, but on the wrong failure.
- Scope against Decisions: each In item maps to one Decision; item 4 lacks a criterion (F6). No Decision or Constraint contradicts an Out item.
