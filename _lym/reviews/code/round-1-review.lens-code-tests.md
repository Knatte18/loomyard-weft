```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test row that pins it.

**Per behaviour, the test that pins it**

1. `refuseUnseededBoard`, origin holds no board, so it names the `--reset` clone: pinned by `TestCloneSingleRepo_FailedSeedPushLeavesNothing/BoardWithNoUpstreamIsNeverPushed/OriginHoldsNoBoard` (clone_singlerepo_integration_test.go:34-37, the message plus `wantReset`).
2. `refuseUnseededBoard`, origin's tip equals the local tip, so it names the four `git config` settings: pinned by `.../OriginHoldsTheSeed` (test lines 38-48). After setting the upstreams, the row also checks that the push succeeds (lines 116-123), which reaches the way forward.
3. `refuseUnseededBoard`, origin's tip is a strict ancestor of the local tip (local board commits made after the killed seed): **none**.
4. `refuseUnseededBoard`, origin holds another board whose commit is missing locally, so `isAncestorOrEqual` returns false with no error and the join is named: pinned by `.../OriginHoldsAnotherBoard` (lines 49-56, 82-84).
5. `refuseUnseededBoard`, origin cannot be read, so it names a rerun once origin answers: **none**.
6. `refuseUnseededBoard`, the local board is unborn (`gitrepo.ErrNoCommits`) and gets the another-board refusal reading "no commits" (commit f7d565cb6): **none**.
7. `refuseUnseededBoard`, reading `PrimeRecordsBranchKey` fails: none. I file no finding, because the clone always writes the key (clone.go:391) and the read cannot reasonably fail.
8. The refusal writes no failed-push record and leaves origin's namespace unchanged: every row of `BoardWithNoUpstreamIsNeverPushed` asserts both (lines 109-114).
9. `pushCloneSeed`, the records push is withdrawn, giving "origin took the records branch … and the seed withdrew it": pinned by `TestCloneSingleRepo_SeedPushKilledBeforeOriginsAnswer/RecordsBranchLanded` (killedpush test lines 81-90).
10. `pushCloneSeed`, the board push is withdrawn, giving "… the seed deleted the board; the seed returns the records branch …": pinned by `.../BoardLanded` (lines 71-80).
11. `pushCloneSeed`, the board push ended unanswered and was not withdrawn while the error was nil (origin holds no board, or another clone's), giving "origin took the records branch, and the board's push ended without an answer; the seed returns …": **none**.
12. `withdrawUnansweredSeedPush`, every keep path returns `withdrawn=false`: pinned by `.../BoardNotApplied` and `.../RecordsBranchNotApplied`, which assert the kept-hub error.

**Findings**

- **MEDIUM**, design. `internal/fabricengine/pushclassified.go:223`: the strict-ancestor branch of the seed-landed case has no test row.
  - The only seed row has the local tip equal to origin's tip (test lines 63-81, no local commit after the clone).
  - A mutant replacing `isAncestorOrEqual` with a tip-equality check passes every row, though the doc comment (pushclassified.go:190) and the summary claim the ancestor case.
  - Fix: add a row that commits on the board after the seed, before the push.
- **MEDIUM**, design. `internal/fabricengine/pushclassified.go:207-210`: the unreadable-origin refusal and its "run the same command again once origin answers" way forward have no row.
  - The table has no row that breaks origin's reachability (test lines 33-57).
  - Fix: add a row that points `remote.origin.url` at a missing path, or removes the bare, before the push.
- **LOW**, design. `internal/fabricengine/pushclassified.go:218-219`: the unborn-board refusal that commit f7d565cb6 added has no row.
  - No test makes the board unborn while origin holds a board (test lines 33-57).
- **LOW**, design. `internal/fabricengine/destroy.go:1783`: the not-withdrawn board wording "origin took the records branch, and the board's push ended without an answer" has no row.
  - The killed-push table reaches only the withdrawn path and the keep paths (killedpush test lines 69-115).

Not a finding: replacing "origin's records namespace is empty" with "unchanged from before the push" keeps the `OriginHoldsNoBoard` row as strict as the old assertion. The old test passing shows `CloneHub` leaves the namespace empty before the push.

Tool calls: none. The context already held everything this lens needed.
