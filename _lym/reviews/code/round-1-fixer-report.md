## Fixer-report, round 1 (code)

All five findings are fixed and committed. `lym gate test` passes at tier 0 and tier 1 for `internal/slackengine`, `internal/slackcli`, `internal/creelcli` and `internal/configcli`; the tier 1 rows for F2 and F3 ran.

## Changes

- **F1** (`internal/slackengine/outbound.go`, `outbound_test.go`, `doc.go`), fixed as proposed, then adjusted after the self-check:
  - `outboxRecord` gains `Stale`, incremented and written in the same `writeOutbox` call that records the skipped id.
  - `post` posts the count line from the record after `postMessages` returns, whatever it returned.
    A line that fails to post keeps its count and fails the read only when the messages themselves posted.
  - The line is held back when the read ended on a rate limit or on a failed record write (self-check findings below).
  - `TestOutboundRead` row `StaleMailCountLineSurvivesAFailedReadAndPostsOnce` has three cases: line posts during the failed read, line fails to post and waits, and line waits out a rate limit.
  - The commit for the F4 test change also carries the first version of this row, because both live in `outbound_test.go` and I made the edits in one round.
- **F2** (`internal/creelcli/delivery_test.go`, `cli.go`, `delivery.go`, `goclient.go`), fixed as proposed:
  - New `TestHubDelivery` row asserts `HubStrands` returns the prime's and the pair's strands over the production wiring.
  - After the self-check, `newProductionHubDelivery` is the one constructor for both `HubStrands` and the daemon's `serveDaemon`, so the row pins the daemon's set too.
- **F3** (`internal/slackcli/wiring_test.go`, new), fixed as proposed: `TestReceiverPaused` over a hubforge hub with a pause entry for the pair only; the hub orch reads not covered and the beam orch reads covered, so a swap of the two worktrees fails both rows.
- **F4** (`internal/slackengine/outbound_test.go`), fixed as proposed: the 5-refusal test is a table with a row where the replacement line itself fails to post; it asserts the read is acknowledged, six Slack calls (no retry), no line accepted and no `refusals` in the outbox file.
- **F5** (`internal/configcli/configcli.go`), fixed as proposed: the `buildConfigLong` hunk is reverted to its previous wording. `TestConfigLong` asserts only words present in both wordings, so no gate required the rewrite.

- **Gate repair** (`internal/slackengine/outbound.go`): the gate found the `outboxRecord` comment I added for F1 fixed-column wrapped, which kept the tests from running.
  I broke it into one sentence per line, one per field, and committed the change; `lym gate test ./internal/slackengine` then passed at tier 0 with all of its targets run.

- **Verify repair 1, F5 reinstated** (`internal/configcli/configcli.go`): the full verify failed `TestCLIForm_EveryCommand` because `lym config`'s long help is capped at 2000 bytes and my revert made it 2013.
  The editor-paragraph rewrite in commit `667e7cac` is the gate's required change, so F5's own exception applies and I restored that wording; the reason is in the commit message.
  This overturns the F5 entry above, and the self-check's "no gate required the rewrite" claim was wrong: I checked `TestConfigLong` and not the byte cap.
- **Verify repair 2** (`internal/slackcli/testmain_test.go`): the full verify failed `TestHermeticGitEnv_GitSpawningPackagesHaveTestMain` because `wiring_test.go` for F3 uses `hubforge.CopyHub`.
  `TestMain` now calls `gitkit.HermeticGitEnv()` before `tmuxkit.Main`, as `creelcli` does.

## Disputed

None.

## Deferred

None.

## Self-check

Forks run: `fix-code-correctness`, `fix-consistency`, `fix-proposal`, `fix-affected-set`. Every fork returned a report; each is saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

- fix-code-correctness, LOW: count line can post twice when the record write fails. Same finding as the fix-proposal one. Fixed as proposed: `post` skips the line when the error wraps `errOutboxWrite`. No dedicated test: the harness has no seam that fails a write after a successful read.
- fix-code-correctness, LOW: extra Slack call straight after a rate limit. Same finding as the fix-proposal one. Fixed as proposed: `post` skips the line on a rate limit, and the third row of the F1 test fails if it posts.
- fix-consistency, LOW: `slackengine/doc.go` omitted the persisted stale count. Fixed as proposed.
- fix-proposal, LOW: record-write duplicate. Merged with the fix-code-correctness finding above.
- fix-proposal, LOW: rate-limit call. Merged with the fix-code-correctness finding above.
- fix-proposal, LOW: the `HubStrands` row did not pin the daemon's own delivery construction. Fixed as proposed with the shared `newProductionHubDelivery`.
- fix-affected-set: no findings.
