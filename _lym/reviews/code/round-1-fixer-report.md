# Fixer report — code review round 1

Every finding is fixed except the call-site part of F3, named under Deferred.
Tier 0 passed for `orchcli`, `beamcli`, `battencli`, `fabricengine` and `pairteardown`.
Tier 1 passed for `orchcli` after the changes below; tier 1 of `beamcli` passed before them, and the later `beamcli` and `battencli` changes were covered by tier 0.

## Changes

- **F1** (`13e75131`): fixed as proposed.
  The run-mail bullet in `pattern/PATTERN-pair-remote.md` now follows the board-settle bullet, which keeps its "A `running` record..." tail.
- **F6** (`bd00f48a`): fixed as proposed.
  The paragraph about dropping unread creel mail sits after the `RemovePair removes the task's pair.` sentences in `RemovePair`'s doc comment.
- **F7** (`eb5efb27`): fixed differently.
  Proposed: add `&& !f.Prime` to the records case of `pairSlug`.
  Not made: `pushclassified.go:269` sets `Prime` only for the code side, so the check would never be true for a records record and would change nothing.
  A records record of the prime does carry a `CodeWorktree`, so it answers the prime's directory name.
  That name is never a pair slug, so no teardown drops the mail.
  Made instead: the `pairSlug` doc states this, and `TestFailedPushRecord` has a row for a records branch with no paired code worktree answering no run.
- **F5** (`050f1953`): fixed as proposed.
  `notices.Post == nil` joins the nil-seam condition in `dobbydeps_integration_test.go`.
- **F3** (`7c561838`, `c21550c2`): fixed with an own change for the beamcli and battencli tags.
  Proposed: assert at the call sites.
  Made: `runNoticeTags` in `beamcli` and `runNotifyTags` in `battencli` carry the tags, each with a row-test, since a call-site test needs a creel daemon and a beam chain in both packages.
  `NotifyParent`'s Run is pinned end to end by `TestCreelNoticeTags/NotifyParentMailIsDroppedWithThePairsSlug`.
- **F2** (`c21550c2`): fixed as proposed.
  `TestCreelNoticeTags` in `internal/orchcli/pushnotifier_integration_test.go` drives `GoClient.PostTagged` and `GoClient.DropRun` against a real in-process daemon.
  It asserts a same-topic post supersedes the earlier unread notice, and a drop returns a run's mail with reason `run-gone`.
  The test helper `startCreelDaemon` now holds the receiver busy, so posted mail stays unread.
- **F4** (`c21550c2`): fixed with an own change.
  Proposed: read the tags on the fake poster in `TestInstallPushNotifier`, which has no fake poster since it runs a real daemon.
  Made: the failed-push notice's Topic is shown by its supersession and its Run by `dropRunMail("task")`.
- **F8** (`c21550c2`): fixed with an own change.
  The closure became the named function `dropRunMail`, which the new test calls and which `InstallRunMailDropper` installs.
  The `TEARDOWN` signing is not asserted: the log's undelivered entry carries no sender, so the test shows only that a program client's drop succeeds.

## Gate repair

The gate flagged two comments that wrapped mid-sentence: the `pairSlug` doc in `internal/fabricengine/failedpush.go` and the `failedPushNotice` helper comment in `internal/orchcli/pushnotifier_integration_test.go`.
Both now hold one sentence per line, and the gate's reports on them are resolved.

## Self-check

- fix-proposal: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-code-correctness: LOW, `TestRunNoticeTags` and `TestRunNotifyTags` pass with a wrong tag at the call sites.
  Disposition: deferred, see below.

## Disputed

None.

## Deferred

- F3, call-site part (self-check LOW): a test that fails when the dobby `runNotices.Post` closure or `notifyRunParent` passes wrong tags needs a creel daemon with a live orch and, for batten, a beam chain, in `beamcli` and `battencli`.
  The review's `fix` key allowed the extracted-function route, so the gap is left open and flagged here.

## Rulings

None.
