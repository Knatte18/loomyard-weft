## Fixer-report, round 1

Every finding was fixed; the gate (`lym gate test`, tiers 0 and 1, over `internal/boardengine`, `internal/boardcli`, `internal/fabricengine` and `cmd/lym`) passes.
Commits are per finding group, since F2, F5, F6, F9 and F10 share files.

## Changes

- F1: `replay.go` gets a `record_body_sha256` row in `replayFields`; `replay_test.go` gets a row where the body and stamp change locally and both land on origin's entry.
- F2: `recordupsert_test.go` gets refusal rows for a filed part, an unfiled project's worktree and live weft, a run-held family project, and a project the store records as filed by another family's project.
- F3: `store.go` carries `RecordBodySHA256` with the discussed flag on a merge that upserts a removed slug again; `decision_test.go` asserts the stamp survives.
- F4: the board skill documents `upsert --record-file` and promote's `warning` key.
  The gate's fabric-vocabulary scan refused the skill's bare "weft branch" wording, so the skill now says "records branch".
- F5: the probe-failure way forward reads "resolve the cause above, then re-run the upsert" in `recordupsert.go`; the engine and CLI tests expect it.
  A `TestCLI` row runs `upsert --record-file` under `--board-path`, expecting the refusal and nothing written.
  The fixed own change is the way-forward wording in the CLI closure's `--board-path` error, which now says to run the verb from a worktree of the hub.
- F6: fixed by stating that stdin is not read, in the flag's help text and in `boardcli/doc.go`; the line the review also implied for the `Long` text was dropped again after a self-check fork showed it exceeds the help size cap.
- F7: a `TestRecordsBranchLive` row with a local weft branch and a missing origin URL expects live and no error.
- F8: a `TestPromoteToRecordWarning` row for a stamped note without the sketch label and an edited body expects no warning.
- F9: `testtiming:keep` directive with its reason added above the family-project test.
- F10: kept the environment skip and stated in the closure comment and `boardcli/doc.go` that it is a test and CI bypass.

## Disputed

Nothing disputed.

## Deferred

Nothing deferred.

## Rulings

No parent ruling reached this round.

## Self-check

- fix-code-correctness, BLOCKING: the added `--record-file` line in upsert's `Long` text likely exceeds the 2000-byte help cap; fixed by dropping the line, and `cmd/lym` tests now pass.
- fix-code-correctness, LOW: F2's rows leave the `filed.Project == slug` clause unpinned; fixed by a row where the store records the project as filed by another family's project.
- fix-proposal, LOW: the F2 filed-part row does not reach the project-equals-slug clause; same fix as above.
- fix-proposal, LOW: the way forward under `--board-path` is not actionable; fixed by the closure's message now naming a hub worktree.
- fix-proposal, LOW: a stamp clash in a replay is reported lost beside the body clash with raw digests; no change, since the stamp must merge with the body it stamps (D5) and the extra line is consistent with how every other field clash reports.
- fix-consistency: no findings.
- fix-affected-set: no findings.
