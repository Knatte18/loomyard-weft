---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md, D7 (per-OS not-empty value, Windows)
    summary: On Windows a late writer still holding its reed lock file makes the leftover removal fail with a sharing violation, not ERROR_DIR_NOT_EMPTY, so D7's retry never fires on Windows in the window it exists for
    fix: In D7, either extend the Windows half of the per-OS sentence to also retry windows.ERROR_SHARING_VIOLATION (a file under the target still open by its writer), under the same attempt count and injected sleep, and add a Windows row to D7's acceptance criterion; or state in D7 that on Windows only a refill after the writer exited is retried, and add an Open risks bullet that a lock file still held during the retry window fails the removal as today on Windows.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md, D7 (Not-empty is recognized by both halves)
    summary: D7 recognizes the first half as git exited nonzero with a delete failure, which no stated signal decides without reading git's message text, the thing the same sentence forbids
    fix: In D7, reword the first half of the recognition sentence to the condition D7 already uses to enter the branch, namely git exited nonzero, the target is no longer a registered linked worktree and the target is still on disk, so neither half reads git's stderr.
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md, D7 and Constraints
    summary: D7 routes the leftover removal through the contained removal directly, which records no mutation, and git's own failure records no worktree removal, so Remove's envelope would not show the deleted directory
    fix: In D7, add that a successful leftover removal appends the same path-removed mutation the gated path removal records for a directory; in Constraints, add PATTERN-mutation-record beside PATTERN-fabric-destruction-chokepoint for D7.
---

### [MEDIUM] D7's Windows not-empty value misses a held lock file

**Location:** `_lyx/discussion/decision-record.md`, D7.

**Issue:** D7 names `windows.ERROR_DIR_NOT_EMPTY` as the Windows value that counts as not-empty, and retries nothing else.
The writers D7 targets create reed's lock files (`reed.lock`, `reed.json.lock`) through `internal/lock`, which wraps gofrs/flock: the file is opened with `os.OpenFile` and held with `LockFileEx` for the operation's length.
Go opens Windows files without `FILE_SHARE_DELETE`, so while the writer holds the file, deleting it fails with a sharing violation, and the contained removal returns that error rather than directory-not-empty.
That is the "any other removal failure" D7 returns at once, so on Windows the retry fires only when the writer has already exited and something else refilled the directory, which is not the race the sighting shows.
D7 states per OS what counts as not-empty, as asked, but the Windows value does not match the Windows failure, and neither Open risks nor Scope says Windows is left as today.

### [LOW] D7's not-empty recognition names a git signal it cannot read

**Location:** `_lyx/discussion/decision-record.md`, D7, the sentence "Not-empty is recognized by both halves".

**Issue:** The first half reads "git exited nonzero with a delete failure", and the sentence ends "never by message text".
`removeGitWorktree` returns a `GitError` carrying only the exit code and stderr, so "with a delete failure" can only be decided from git's message, while D7's branch condition earlier in the Decision (nonzero exit, no longer registered, still on disk) already decides it without text.
A plan writer reading the recognition sentence literally would parse stderr.

### [LOW] D7's leftover removal records no mutation

**Location:** `_lyx/discussion/decision-record.md`, D7 and `## Constraints`.

**Issue:** `removeGitWorktree` appends `KindWorktreeRemoved` only on success, and `removeContainedPath` appends nothing; only the gated `removePath` appends `KindPathRemoved`.
D7 has the leftover removed "through the contained removal" because the gated path's registered-linked-worktree ownership check would refuse an unregistered target, so as worded, a Remove that deletes the task worktree directory on the retry path reports no removal of it in its mutation record.
`PATTERN-mutation-record` requires a mutating fabric verb to fill its record, and D7's Constraints line names only the destruction chokepoint and pair teardown.

## Focus verdicts

- D7: the retry sits on the branch the failure reaches, the not-a-linked-worktree branch after a `GitError`, which reads the live worktree list through `isRegisteredLinkedWorktree`.
  The premise holds: `git worktree remove` deletes the work tree, continues past a failure, and deletes the registration.
  The ownership proof is stated and sound, since `checkPathRequest` runs the registered-linked-worktree and dirtiness gate before git is invoked.
  The acceptance criterion starts from `git worktree remove` failing.
  The per-OS values and the recognition wording are F1 and F2.
- D1: not a defect.
  Both signals it accepts, a session start and a turn start, are `SessionSignalSessionStart` and `SessionSignalTurnStart`, which `claudeengine`'s `ParseSessionSignals` produces from the recorded `SessionStart` and `UserPromptSubmit` hooks; D1 reads them through the session-signal parser, not `EventKind`.
- D2: not a defect.
  Its Bound names the bypassed guard (the verified send's landing check), the proof that replaces it (a turn end in the events file), and the bound on a turn that never starts or never ends (the skill-load timeout measured from the end of `awaitStartup`).
  A turn that never starts and leaves a blank pane is bounded earlier by the startup deadline, which D1's second acceptance criterion covers.
