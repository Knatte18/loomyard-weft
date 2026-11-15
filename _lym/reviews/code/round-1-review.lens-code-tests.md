```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I named the test row that would fail if the behaviour broke.

**Coverage list, per behaviour**

1. Snapshot taken at spawn and stored in `attempt.json`: pinned by `TestCall_Edits`, the attempt-snapshot switch at `internal/shedarbiter/call_test.go:580-585`.
2. Spawn refused when the worktree can't be read, with nothing spawned: pinned by `TestCall_Edits` "a worktree that cannot be read at spawn…" (`call_test.go:516`).
3. Resume with no commit: pinned by `TestCall_Edits` row 1.
4. Resume after commits with a goto, commits listed in the envelope and `envelope.json`: pinned by `TestCall_Edits` row 2 (`call_test.go:485`, 558, 572-575).
5. Resume after commits with no goto is handed on as `precondition`, and the question names the commits: pinned by `TestCall_Edits` row 3 (`call_test.go:489`).
6. Agent's own hand-on keeps its question and appends the commits: pinned by `TestCall_Edits` row 4.
7. Uncommitted changes, branch switch and rewritten history each hand on as `precondition`: pinned by `TestCall_Edits` rows 5–7.
8. Timeout after commits names them: pinned by `TestCall_Edits` row 8.
9. Worktree read fails after the arbiter (covers detached HEAD, because `CurrentBranch` errors on it): **none**.
10. `IsAncestor` or `CommitLog` fails after the arbiter: **none**.
11. Uncommitted changes together with commits, where the hand-on keeps the commits (`edits.go` problem branch, `call.go:327-328`): **none**. The uncommitted-changes row has no commits and `wantCommits` is nil.
12. Agent hand-on over a worktree problem appends "Check the worktree first: …" (`edits.go` `appendTo`, problem note): **none**.
13. `MayEdit` wired from `Deps.Worktree`, choosing the fix or no-fix fragment: pinned by `TestCall_Edits` rows 1 and 10 (`wantPrompt`) and by `TestPrompt`.
14. `editScope` fills `worktree` and `run_id` into the fix fragment: pinned by `TestPrompt` "every marker value…" (`prompt_test.go:832`).
15. Fragment markers: pinned by `TestStencils_TopLevelMarkers` (`markers_test.go:45-46`).
16. `Seam` maps `envelope.Commits` into `ArbiterOutcome.Commits` (`seam.go:40`): **none**. `TestCall_Edits` stops at the call's envelope, `TestGoDriver_Settle` builds `ArbiterOutcome` directly, and no test calls `Seam(…).Judge`.
17. `lym loom arbiter` output gains `commits`, printed as `[]` when nil (`internal/loomcli/arbiter.go:166-177`): **none**. No loomcli test references `arbiterEnvelopeFields`.
18. `arbiterWorktreeSeam` wiring through fabric: **none**. It is production wiring, so this is noted only.
19. Re-step settle record carries `commits`: pinned by `TestGoDriver_Settle` "an arbiter resume after its own commits…" (`drive_test.go:432-436`, 987-997).
20. Stop report's attempt line carries `commits`: pinned by `TestGoDriver_Settle` "an arbiter hand-on after its own commits…" (`drive_test.go:424-429`).
21. `SettleRecord.Commits` JSON key: pinned through `TestGoDriver_Settle`'s re-step row, which reads the records back. `TestParseSettleRecords` only switched to `reflect.DeepEqual` and has no commits row, which is acceptable.

**Findings**

1. **MEDIUM, design.** `internal/shedarbiter/edits.go` `readEdits`: three failure paths have no test row. They are the post-arbiter `Snapshot` failure (this is also the detached-HEAD case), the `IsAncestor` error and the `CommitLog` error, each of which hands on with cause `precondition`. `fakeWorktree` (`call_test.go:420-425`) never fails `IsAncestor` or `CommitLog`, and `snapshotErr` applies to the spawn read too, so no row reaches the post-arbiter read error. These are error paths at a trust boundary, an unavailable dependency, so the testing rules require them. Evidence: `call_test.go:412-425`, 516; `edits.go:666-685`.
2. **LOW, design.** `call.go:327-328` and `edits.go` `appendTo`: two outcomes have no row. One is a resume over uncommitted changes after commits, which must still list the commits in the envelope and question. The other is an agent's own hand-on whose question gains the "Check the worktree first" note. Evidence: the `call_test.go:499` row has no commits, and no row combines `handOnVerdict` with a worktree problem.
3. **LOW, design.** `internal/shedarbiter/seam.go:40`: no test fails if `Commits: envelope.Commits` is dropped, which would silently drop commits from the driver's settle record and stop report. Evidence: `seam.go:40`; no `_test.go` in `internal/shedarbiter`, `loomcli` or `battencli` calls `Seam(…).Judge`.
4. **LOW, design.** `internal/loomcli/arbiter.go:166-177` `arbiterEnvelopeFields`: the new `commits` output key, and its empty-list rendering, have no test. Evidence: no loomcli `_test.go` references `arbiterEnvelopeFields` or a `"commits"` key.
5. **LOW, consistency (test economy).** The redundancy report lists `TestPrompt` (`internal/shedarbiter/prompt_test.go:32`) as contained by `TestCall_Edits`. Its assertions pin what `TestCall_Edits` never checks: every marker value, absent inputs reading `none`, the parent and edit directives, the worktree path and the `Arbiter-Attempt: <run-id>` trailer filled into the fix fragment. It should keep a `//testtiming:keep <reason>` line directly above `func TestPrompt`, naming those pins. Evidence: `prompt_test.go:29-32` has no keep marker; `call_test.go:569-571` checks only the heading.
6. **NIT, consistency (test economy).** `TestPrompt`'s new fix/no-fix heading assertions (`prompt_test.go:815`, 834, 842-843) and `TestCall_Edits`'s `wantPrompt` rows (`call_test.go:482`, 521, 569-571) both assert which fragment renders, from the same flag one layer apart. Only the `MayEdit` wiring is unique to `TestCall_Edits`, so the duplicate fragment-selection check could live in one of the two.

Tool calls:
1. Grepped `settle_test.go`, the `sheddriver` tests and `internal` tests for commits rows and for tests of `arbiterEnvelopeFields` or `arbiterWorktreeSeam`, to confirm whether the settle-record key, the verb output and the wiring had covering tests; my context lacked those test bodies.
