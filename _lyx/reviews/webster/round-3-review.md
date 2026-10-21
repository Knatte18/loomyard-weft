---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/reedcli/watchdog_integration_test.go:344
    summary: TestWatchdogDaemon/DownThenUpDoesNotKillDaemon still fails the publish verify, because the widened idle window of three seconds barely exceeds the 2.59s the failing run took
    fix: In the shared-loop setup, replace the literal `timing.IdleCycles = 15` with a window derived from the test's own wait budget, `timing.IdleCycles = int(watchdogTestWait / watchdogTestCycle)`, so the window is ten seconds. No subtest of that loop depends on its idle exit, since each ends by cancelling `loopCtx`. Drop the "fifteen cycles, three seconds" wording from the comment above it and keep only the reason.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: PATTERN.md:11
    summary: Thirteen overview entries other than the two declared ones differ from main's text, so the overview no longer reads as on main
    fix: In PATTERN.md restore main's text, taken from `git show main:PATTERN.md`, for `PATTERN-durable-vs-ephemeral-state`, `PATTERN-hub-suffix`, `PATTERN-push-both-sides`, `PATTERN-recipe-setting-lookup`, `PATTERN-transient-stop`, `PATTERN-driver-choice-single-site`, `PATTERN-batcher-registry`, `PATTERN-creel-store`, `PATTERN-stencil-ownership`, `PATTERN-model-choice-mapping`, `PATTERN-pane-binary-resolution`, `PATTERN-edit-directive` and `PATTERN-testkit`. Remove the `(test)` and `[background]` link changes along with them. Delete the new `pattern/PATTERN-recipe-setting-lookup.md`, or keep it only if main's entry text is restored with its link left as on main. If the overview then exceeds its headroom line, make room by shortening the text of the two declared entries or by raising the headroom, never by editing an undeclared entry.
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/fabricengine/destroy.go:1793
    summary: When the tip restore after a failed replay is itself refused, the replayed local commits are named only by error text and the reflog, not by a tag
    fix: In `restoreReplayTip` and in `resetBack` inside `cherryPickOnto` (both destroy.go, with the `replayBolt` call site in replay.go), when `ResetKeep(startTip)` fails, first tag the start tip through `setAsideTag(rec, repo, "restore-failed", startTip)`, which routes through the same chokepoint, and put the tag name in the returned error with the way forward `git reset --keep <tag>`. Both functions take `rec` as an added parameter. Add a row to the replay integration tests that holds a refusal of the restore and asserts the tag names the starting tip.
  - id: F4
    severity: LOW
    class: design
    location: internal/fabricengine/destroy.go:1857
    summary: A failed read of the final tip after a completed pick returns an error with the branch left on upstream plus the picks, not on its starting tip
    fix: In `cherryPickOnto`, replace the bare `return nil, err` after the final `repo.CurrentSHA()` with `return nil, errors.Join(err, resetBack(false))`, as every other failure after the reset does.
---

### [BLOCKING] DownThenUpDoesNotKillDaemon's window is still thin

**Location:** internal/reedcli/watchdog_integration_test.go:344
**Issue:** The publish verify failed on this test after 2.59s (`.lyx/verify/verify-7.log`), with the three-cycle production window of 600ms.
The branch widens that window to fifteen cycles of 200ms, three seconds, for the shared loop only, and the other watchdog tests keep their timing.
The failing subtest ran 2.59s end to end, including a down and an up under a loaded package run, so the gap the daemon must outlast was somewhere under that.
A window of three seconds clears it by about 0.4s, which is no margin for a test that flaked once under load.
Nothing in the shared loop's other subtests needs it to idle-exit, so a window matching `watchdogTestWait` costs nothing.
The comment at line 376 also still says `watchdogHubIdleCycles` covers this gap, which the widened loop no longer uses.

### [MEDIUM] Overview entries differ from main without a declaration

**Location:** PATTERN.md:11
**Issue:** `git diff main..HEAD -- PATTERN.md` shows thirteen entries besides `PATTERN-origin-reconcile` and `PATTERN-push-failure-visible` with shortened text, and two of them (`PATTERN-hub-suffix`, `PATTERN-testkit`) with changed wording.
The task's declared changes list only the two.
Agents get the overview inlined in their prompt, so a qualifier dropped from it is dropped from what they judge by: `PATTERN-edit-directive` loses "orch stencils are exempt", `PATTERN-durable-vs-ephemeral-state` loses "nothing a run needs lives only under `.lyx`", and `PATTERN-hub-suffix` loses "no code recognises retired `-HUB`".
The background files still carry most of the detail, but the overview is the authoritative line.
Commit 8d6d07b2 restored these entries and commit 7edad217 cut them again to fit the headroom line, so this finding recurs from the round-1 fix rather than being new.

### [MEDIUM] A refused restore leaves replayed commits named only by error text

**Location:** internal/fabricengine/destroy.go:1793
**Issue:** The set-aside commits are tagged before the branch moves, but the other local commits are not.
When `restoreReplayTip`, or `resetBack` in `cherryPickOnto`, cannot put the starting tip back, the branch stays on upstream and the starting tip is named in the returned error text and the reflog only.
The error's own wrapped message is `restore <sha> after a failed ...`, with no ref to hold the commits against a gc.
The existing rows cover a refused reset before the branch moves and a failed by-entry apply with a successful restore, and no row pins a refused restore.

### [LOW] The final tip read leaves the branch moved

**Location:** internal/fabricengine/destroy.go:1857
**Issue:** Every error return after the reset uses `resetBack` except the last `CurrentSHA`, which returns the bare error.
The replay has finished by then, but the caller reads the error as a failure, and the originals of the replayed commits are then reachable only through the reflog.

## Checked, no finding

- `replayWeft` and `replayBolt` tag every classified set-aside commit before the branch moves, and a by-entry set-aside is tagged before its commit; the tagging-failure and by-entry-failure rows pin both.
- `cherryPickOnto` wraps `ErrReplayDirty` only around the dirtiness refusal and `IsLocalChangeRefusal`, and a held index lock surfaces as a named git failure, pinned by the index-lock row.
- `settleUnmovedWeft` returns a failed local read, warns only for a missing upstream or nothing behind, and so never returns success with `weft_pulled` false on an infrastructure error.
- The two declared PATTERN entries land with their declared text.
