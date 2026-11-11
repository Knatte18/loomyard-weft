---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: pattern/PATTERN-pair-remote.md:70
    summary: The new run-mail bullet splits the board-settle bullet, so its last two lines now read as part of the run-mail bullet.
    fix: In pattern/PATTERN-pair-remote.md move the three-line bullet "After a successful removal the composite drops the hub's unread creel mail..." to after the line "The composite keeps the claim and warns with `lym batten cancel <slug>`." (line 74), so the board-settle bullet keeps its "A `running` record..." continuation.
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/creelengine/server.go:305
    summary: The wire path for post tags and the drop-run verb has no test; a dropped field would silently disable supersede and teardown drop while every Store-level test passes.
    fix: Add one test that drives a real daemon or server dispatch connection (or the creelcli journal-exists path) through GoClient.PostTagged then a second PostTagged under the same topic, and GoClient.DropRun for a tagged run, asserting the older mail is returned undelivered with reasons superseded and run-gone; put it in the existing tier that allows a local daemon.
    origin: lens:code-tests
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/orchcli/notify.go:696
    summary: The run tags the callers set (NotifyParent Run, battencli notifyRunParent Run, beamcli run-notice topic) have no test, so a wrong or empty tag would disable supersede and teardown drop unnoticed.
    fix: Add rows to the existing orchcli notify test asserting that NotifyParent posts with Run equal to the pair's WorktreeName, and a row for the beamcli runNotices.Post closure (or a small extracted topic function) asserting Topic "dobby-run-notice/<slug>" and Run "<slug>"; assert battencli notifyRunParent passes Run slug through NotifyRunFrom in the existing battencli notice test.
    origin: lens:code-tests
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/orchcli/pushnotifier.go:43
    summary: InstallPushNotifier forwards notice.Topic and notice.Run to the post, but no assertion reads the tags a posted failed-push notice carries.
    fix: In TestInstallPushNotifier (internal/orchcli/pushnotifier_integration_test.go) assert that the mail posted for a notice carrying a Topic and Run arrives with those same tags, using the existing fake poster.
    origin: lens:code-tests
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/beamcli/dobbydeps_integration_test.go:115
    summary: The every-seam-filled check omits the new RunNotices.Post, and a nil Post makes dobby send no run notices without a warning.
    fix: Add `notices.Post == nil` to the nil-seam condition at internal/beamcli/dobbydeps_integration_test.go:115 so a wiring regression fails the test.
    origin: lens:code-affected-set
  - id: F6
    severity: LOW
    class: consistency
    location: internal/pairteardown/teardown.go:432
    summary: The new paragraph sits in RemovePair's doc comment before the sentence that opens with the symbol name, leaving that sentence mid-comment.
    fix: In internal/pairteardown/teardown.go move the paragraph "A successful removal drops the hub's unread creel mail..." to after the "RemovePair removes the task's pair." sentences, so the comment's self-reference stays ahead of it.
    origin: lens:code-affected-set
  - id: F7
    severity: LOW
    class: design
    location: internal/fabricengine/failedpush.go:186
    summary: pairSlug's records case does not check Prime, so a prime's records branch with a CodeWorktree answers the prime's directory name as Run although the doc says it answers empty.
    fix: Check what pairedCodeWorktree sets for the prime's records record; if it fills CodeWorktree there, add `&& !f.Prime` to the records case of pairSlug, otherwise leave the code and add a pairSlug row for a records branch with an empty CodeWorktree answering no run in TestFailedPushRecord.
    origin: lens:code-correctness
  - id: F8
    severity: LOW
    class: design
    location: internal/orchcli/runmail.go:19
    summary: The InstallRunMailDropper closure (TEARDOWN program client calling DropRun and returning the count) has no test.
    fix: Add a row to an existing orchcli test, using the notifier fake seam, that installs the dropper and checks it signs as TEARDOWN, calls DropRun with the slug and returns the dropped count; skip it if F2's wire test already drives it.
    origin: lens:code-tests
---

### [BLOCKING] Run-mail bullet splits the board-settle bullet in PATTERN-pair-remote.md

**Location:** pattern/PATTERN-pair-remote.md:70
**Issue:** The new bullet was inserted before the last two lines of the board-settle bullet. Those lines ("A `running` record with no live registered batten process means the run is gone...") now read as part of the run-mail bullet. The rule document states a false rule and the settle rule loses its tail.

### [MEDIUM] No test of the wire path for post tags and drop-run

**Location:** internal/creelengine/server.go:305
**Issue:** The `TestStore_*` tests call the store directly and the creelcli test never connects. If `server.go` or the client dropped the Topic or Run field, supersede and teardown drop would do nothing and every test would still pass.

### [MEDIUM] Caller-set run tags are untested

**Location:** internal/orchcli/notify.go:696
**Issue:** `NotifyParent`'s Run, battencli's `notifyRunParent` Run and beamcli's run-notice topic are what make the dropped and superseded mail match, and no row reads them.

### [MEDIUM] Failed-push notice tags are not asserted after the notifier hop

**Location:** internal/orchcli/pushnotifier.go:43
**Issue:** `TestInstallPushNotifier` was only retyped; no assertion reads the Topic and Run of the posted mail.

### [MEDIUM] Seam-filled check omits RunNotices.Post

**Location:** internal/beamcli/dobbydeps_integration_test.go:115
**Issue:** `deliverRunNotice` returns false on a nil Post, so a wiring regression sends no run notices and the check stays green.

### [LOW] RemovePair doc comment order

**Location:** internal/pairteardown/teardown.go:432
**Issue:** The self-referencing "RemovePair removes the task's pair." sentence now follows the new paragraph; the surrounding paragraphs already order it oddly, but the diff adds to it.

### [LOW] pairSlug may name the prime for a records branch

**Location:** internal/fabricengine/failedpush.go:186
**Issue:** The doc says a records branch with no paired code worktree answers empty, but the records case does not check Prime and the lens read `pairedCodeWorktree` as filling `CodeWorktree` on every records record. Unverified by the handler: check before changing.

### [LOW] InstallRunMailDropper closure untested

**Location:** internal/orchcli/runmail.go:19
**Issue:** The `TEARDOWN` signing and the count it returns are pinned by no test.

## Rejected

- Lens code-correctness finding 2, a failed supersede append after the post is journaled returns an error for a stored post: the sibling failures in the same locked callback behave the same way, and a journal append failure is not a state to build recovery for.
- Lens code-correctness finding 3, one topic per run lets a newer run notice replace an unread one of another condition: the amended entry asks that repeats be superseded and a changed state still reach the orch, and the newest notice does reach it.
- Lens code-correctness finding 5, a teardown run from a non-orch strand's pane drops nothing: creel's refusal is documented in the code and the package doc, and teardown logs a Warn.
- Lens code-tests NIT, the two remote-teardown tests check only the line: the topic is set in one function that `TestNoticeRemoteTeardown` already pins.
- Test economy: every redundancy-report candidate is "not judged", and no new test duplicates another's coverage.
