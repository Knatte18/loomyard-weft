# Round 1 fixer-report

## Changes

- F1 (MEDIUM): fixed as proposed, with one narrowing.
  `internal/reedengine/bootwindowlog_test.go` drives `logBootWindowLocked` against the scripted fake tmux and the package's `logcapture`.
  Rows: a three-field answer logs `window=@0`, `pane=%1` and `name=bash`; a name with a space stays one field; a failed read and a single-token answer log no Info line.
  The review proposed also asserting `booted=true` through `ensureServerAndSessionLocked`.
  That was not done: `lifecycle_test.go` documents the full boot as out of hermetic reach, and `logBootWindowLocked` returns nothing, so the helper cannot affect the boot by construction.
- F2 (LOW): fixed as proposed.
  `internal/reedengine/doc.go` now says an own window, a break-out and a detached window are opened with a name, and that the boot's first window starts named after its shell and is renamed `shell`.
  The line above it now says the pane id is logged where the open reports one, which matches the detached-window line.
- F3 (LOW): no change, per the review's own fallback.
  No existing test in the package drives `openOwnWindowLocked` or `moveShellWindowStrandsLocked` with a log capture, and the review forbids adding a harness for this finding alone.
  The `name` field on those two lines is log-only.
- Self-check follow-ups: the failed-read row now carries a well-formed answer with the error, so only the error check can suppress the Info line;
  the test was renamed to say what it pins; `doc.go` "A fresh boot's one window ends up named that way."

- Gate repair: the test's doc comment wrapped mid-sentence, so no test ran.
  It now has one sentence per line.

Commits: F1, F2, one follow-up commit for the self-check changes, and one for the gate repair.
The tier-0 gate for `./internal/reedengine` passed after the last change.

## Self-check

- fix-code-correctness, LOW: the failed-read row passed even without the `err` check. Fixed as proposed.
- fix-consistency, LOW: `doc.go` "A fresh boot names its one window that way" contradicted the new wording. Fixed as proposed.
- fix-consistency, LOW: test name and comment claimed more than the rows check. Fixed as proposed (duplicate of the fix-proposal finding below).
- fix-proposal, LOW: same test-name overclaim. Merged with the fix-consistency finding above and fixed.
- fix-affected-set: no findings.

- Second gate failure: `internal/hibernateengine` `TestCarrier` failed once in the verify run with
  `testing.go:1464: TempDir RemoveAll cleanup: ... directory not empty`.
  This round changes nothing in that package (`git diff main...HEAD -- internal/hibernateengine` is empty).
  Rerunning `lym gate test ./internal/hibernateengine` passed (94 of 94), and the same verify log shows it `ok` on later runs, so it is a cleanup race and no change was made.

## Disputed

None.

## Deferred

- The `hibernateengine` `TestCarrier` cleanup race: unrelated to this round's change set, passes on rerun, and its fix belongs to that package's owner.
