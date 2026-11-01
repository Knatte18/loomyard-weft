# Round 1 fixer-report

## Changes

- F1 (decision): left to the parent, who recorded the decision in the decision record. No change.
- F2: `replay` writes `delivered` only when both output copies succeeded; `copyRunFile` returns its error. Test `TestFrontReplay_DeliveredMark`. Commit 863b07b3.
- F3: the runner's lease loop cancels on `leaseLapsed` (read and past, or absent); the startup refusal keeps `leaseHolds`. Test `TestDetachRun_LeaseLapsed`.
- F4: `runRequest` captures `callCtx.Err()` before `cancelCall` and abandons only when the cancel came while the call ran.
- F5: `claim` keeps its exclusive create; every reader of the process file goes through `readProcessRecord`, which reads a present but undecodable file again up to five times, 20 ms apart, before it counts it absent. Tests `TestDetachRun_Claim` and `TestDetachRun_ReadProcessRecord`. The first version linked a temporary file with `os.Link`, which `PATTERN-fs-links` forbids outside `internal/fslink`, so the gate failed; the reviewer's second option replaced it.
- F7: `replayedBusyLine`, `abandonedLine` and `answerRunGoing` take the request's verb; `readRequest` refuses a card-verify request without Args (row `CardVerifyWithoutCard`).
- F3, F4, F5 and F7 share one commit, 6bfa7555, because their edits interleave in the same hunks of `detachrun.go`.
- F6: `pruneOldRuns` removes run directories older than 14 days with no live runner when a front starts a run. Test `TestFrontPruneOldRuns`. Commit 53c2ada3.
- F9: `docs/overview.md` names `keep_warm_cap` and `keep_warm_cap_roles`. Commit 28fda4f5.
- Self-check fixes: the front's other error messages name the request's verb; `doc.go` describes the claim, the lease rule, the delivered rule and the pruning.

## Disputed

- F8, `premise false`: the two in-process branches have rows.
  `test_integration_test.go` "a free slot runs the tier under its env, exits with its code and records the run" sets a strand and `helperCapEnv=0` and asserts exit 7 and the gate's own pid as `Enclosing`.
  Without the zero-cap branch the call would be capped and answer 75.
  "an inherited held slot runs without acquiring" sets a strand and an inherited slot and asserts the held slot and an in-process watchdog log.

## Deferred

Nothing.

## Self-check

- fix-consistency, LOW, `doc.go` claim sentence: fixed.
- fix-consistency, LOW, `doc.go` lease cancel rule: fixed.
- fix-consistency, LOW, `doc.go` delivered rule and pruning: fixed.
- fix-consistency, LOW, `doc.go` abandoned line naming only `gate test`: fixed (the abandoned-line sentence names the verb).
- fix-affected-set, LOW, six front messages saying `gate test:` for a card-verify front: fixed.
- fix-proposal, LOW, same leftover messages: merged with the above, fixed.
- fix-proposal, LOW, `os.Link` unavailable on a filesystem without hard links: moot, since `os.Link` was replaced by the re-read (see F5).
- fix-proposal, LOW, mode 0600 of `process.json`: merged with the code-correctness finding below.
- fix-code-correctness, LOW, mode 0600: moot, since the claim creates `process.json` as before, mode 0644.
- fix-code-correctness, LOW, F5 test passes with the defect in place: no test added for the race. `TestDetachRun_ReadProcessRecord` pins the undecodable and absent cases, and the review asked for none.
- fix-proposal, LOW, F2 partial copy replays twice: fixed as proposed, no change. This is the trade-off the finding chose, a repeated replay against a lost result.
- fix-proposal, LOW, F3 permanently unreadable lease never cancels: fixed as proposed, no change. The finding asks for exactly this, and a corrupt lease file has no writer left but a front's rewrite.
