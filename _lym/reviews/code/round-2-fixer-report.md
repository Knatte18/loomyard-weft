---
dispositions:
  - id: F10
    disposition: fixed
---

# Round 2 fixer report

## Changes

All in `internal/loomcli/prreview_mail_integration_test.go`.

- F10 (`3e73ca9a8`): `DirtyPaths`, `CommitsMissing`, `TaskHasCommits`, `PauseCovered`, `RecordChanges`, `RecordPath` and `RecordLockPath` come from the wiring over the pair.
  The setup commits a file on the pair's HEAD, so the wired `TaskHasCommits` reads true.
  The push counter still wraps the fixture's `PushTask`.
- Self-check (`d889bf5b5`): `DriftVerdict` comes from the wiring and `EnginesRun` from the replayed session's runner.
  The test sets `GIT_ALLOW_PROTOCOL=file`, `GIT_TERMINAL_PROMPT=0` and an empty `GIT_ASKPASS`.
  The header and the `prReviewMailDeps` doc name the config reads again.

Verification: `lym gate test --tier 1 ./internal/loomcli` passed (228 passed, 0 failed, 292 skipped), and the filtered run showed `TestPRReviewGateFailureMail` and its rows running and passing.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal, LOW (comments dropped the mode read): fixed.
- fix-affected-set, MEDIUM (`DriftVerdict`, `EnginesRun` canned): fixed.
- fix-affected-set, MEDIUM (`CommitRecord` canned): disputed, every fix worse.
  The wired seam commits and then pushes the pair (`internal/loomcli/landingdeps.go:457-503`), and the pair's origin must be a GitHub URL for the row to decide on a pull request.
  An uncommitted trial that wired it reached github.com and hung on a credential dialog.
  With the transport blocked, its push fails the row.
- fix-affected-set, LOW (`BaseCommand`, `GateTiers` canned): disputed, every fix worse.
  They belong to the scripted gate command the review accepted.
  A trial that wired both made every row fail: the mail and halt reason lost ` at tier 1`, since the fixture hub's gate config yields no tier for this gate.

## A trial that reached the network

During the self-check fixes, a trial built the test's deps wholly from the wiring.
Its test run asked the real github.com for the pair's origin, and a credential dialog opened on the operator's desktop.
The parent reported it (creel #3269) and was answered.
The trial was reverted and is in no commit.

## Disputed

No review finding was disputed.
The disputed self-check points are listed under Self-check.

## Deferred

Nothing deferred.

## Rulings

None.
