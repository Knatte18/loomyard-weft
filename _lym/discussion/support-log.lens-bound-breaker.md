```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked each claim, bound and guarantee in the decision record against the code one hop past what it cites.

**Findings**

1. **BLOCKING — D2 and D3, ordering and completeness.** A pusher in a separate process can rewrite the records worktree while the driver commits into it.
   - Evidence: on a rejected records push, `retryWeftPush` (`internal/fabricengine/pushclassified.go`) "replays the weft worktree's local commits onto origin's". It try-acquires the weft write lock and defers on a held lock or on an uncommitted change on a path the replay touches.
   - Today the driver pushes in its own process right after its commit, so no commit runs during a replay.
   - Under D1 and D2 the driver keeps committing (`CommitAnchoredPathsWithTrailers`) while the pusher replays. Example: an orch's `lym loom decision add` lands on origin's records branch and rejects the push, and the driver commits a transition mid-replay.
   - Nothing in the record or my reads shows that the transition commit takes the weft write lock. Without it, a transition commit can be lost or reparented.
   - The claim "origin receives the transitions in commit order" also fails under a replay: the commits are rewritten onto upstream, with new SHAs.
   - The record must state the lock that excludes a transition commit from a replay, or rule replays out of the pusher.

2. **MEDIUM — D6 and D3.** An owed or refused push can still stop the run between passes.
   - Evidence: `runfsm` keeps `PushOwed` as an act-rule input, and `ReadRun` fills it from failed-push records. `cancelAside` sets `PushOwed: true` so that a cancel can get past it, which means the act rule refuses on it otherwise.
   - D3 still writes a failed-push record on a refused push. The dobby's next full pass, or a hand pass, then reads `PushOwed` and is refused.
   - So "the run goes on while its push is owed" holds only within one pass. D6 should say which verbs still refuse on `PushOwed` and which no longer do.

3. **MEDIUM — D1 and D4, "the local pause gate".** The seam has one gate, `Admit: pauserecord.Admit("transition push")` (`internal/loomcli/wiring.go`). `926b5c28e` calls it the network-gate admission.
   - D1 drops the network admission yet "keeps the local pause gate". D4 relies on that gate to meet the pusher's pause intent "at its next transition".
   - The record does not establish that a separate local gate exists, or which component honours a pause intent at a row boundary. D4's bound rests on it.

4. **MEDIUM — D4's bound of "at most one row".** The pusher reconciles only right before a push.
   - Counterexample: origin is unreachable, so the pusher sits in the owed backoff (up to 60 s, without end). No reconcile runs, and the run walks any number of rows.
   - Also, the reconcile today sits inside the commit: `check.guardedCommit` returns `errPushStopped`, and the commit is then not made. Under D4 the stopped or owned-elsewhere run keeps committing locally.
   - The bound holds only while origin is reachable. The record should say so.

5. **MEDIUM — D3, "Worktree-Teardown keeps its archive push … which carries any commit the pusher had not pushed".** Per `PATTERN-pair-remote`, a refusal of the teardown's archive push "is logged only, since the removal archives and deletes the branch anyway".
   - Counterexample: origin is unreachable at teardown. The pair's last loom transitions, such as Friction-Reflect and the run's done, are not on origin, and the pair is removed. Those commits survive only in local archive tags.
   - Before this change every transition had already been pushed, so a failed archive push lost nothing.
   - This breaks the goal "in order and complete". Teardown ending the pusher first (D3) widens the gap.

6. **MEDIUM — D3, the dobby restart covers pairs only.** Batten's records live in the prime's records branch, since the batten Shed is anchored at the prime and the prime pushes records only.
   - Counterexample: teardown's `EndSubjectProcesses` ends the slug's batten pusher. The terminal Worktree-Teardown commit's new pusher then fails, or the machine sleeps.
   - The pair is gone, so the dobby's "pair ahead of its tracking ref" check never fires. No resume follows a done run. The prime's records stay ahead until some unrelated commit.

7. **LOW — D2, "at most one pusher per run" and its liveness check.** One slug has two seams: batten's, on the prime records, and the inner loom run's, on the pair.
   - Both seams' processes register under the same slug, and `HandOver`'s `EndSubjectProcesses(prime, slug, …)` ends every process of that slug.
   - A liveness check keyed by slug would let batten's live pusher stop the loom seam from starting one, leaving pair commits waiting until it exits.
   - The record should key the pusher per seam and side, and state how hand-over spares the pusher while ending the other one.

8. **LOW — D5, "resumes the row that record names".** Records are pushed before the code side.
   - Counterexample: a pusher killed between the two sides leaves origin's record naming a `Warp-SHA` origin lacks. The adopting machine's `settleCodeTip` waits `runfsm.CodeTipGrace`, then `ReturnToPreviousTransition` rewinds the run one transition.
   - This path existed before, but continuous pushing hits the window on every push. D5 should mention it.

9. **LOW — D5, "removes the local pair".** Loom's `Push` passes `InsidePair: true` (`internal/loomcli/wiring.go`). `HandOver` then leaves the pair standing "for its parent's dobby to remove".

**Claims attacked**

- Held: per-transition board status writes do not wait for a push. `SetRunStatus` spawns a detached sync (`internal/boardengine/board.go`), so D1's "pushes nothing" survives the board write.
- Held: the takeover check runs inside the rejected-push path, at the start of `retryWeftPush`, before the replay.
- Held, with a gap: the hand-over path is reusable from the pusher. Its self-exclusion is not shown (finding 7).
- Broken: commit order and completeness under replay (finding 1).
- Broken: the run goes on while its push is owed (finding 2).
- Broken: the pause gate (finding 3).
- Broken: D4's bound of one row (finding 4).
- Broken: the teardown archive push carrying leftovers (finding 5).
- Broken: the dobby restart's coverage (finding 6).
- Partly broken: one pusher per run (finding 7).
- Incomplete: the takeover resume point (finding 8).
- Inaccurate: the hand-over removes the pair (finding 9).
- Held: an older binary's marker. `push_pending` only stops forcing a push, and the tolerant read is kept.
- Not attacked: the Finalize double-act guard, already listed under Open risks.

**Tool calls**

1. `internal/fabricengine/pushclassified.go`, `retryWeftPush`: to see what a rejected push does, a replay plus the takeover check, beyond the subagent's summary.
2. `internal/pairteardown/handover.go`, `HandOver`: to check process ending and pair removal, since the record leans on it.
3. `internal/battencli/codetipsettle.go` and `internal/fabricengine/codetipsettle.go`: the missing-code-tip path, to check D5's resume-point claim.
4. `internal/loomcli/wiring.go`, `newCommitStatusSeam`: to see where the reconcile sits (`guardedCommit`) and what `Admit` is.
5. A grep for `SetBoardStatus` and `errPushStopped`: to find the per-transition board write and the reconcile's call sites.
6. `internal/loomcli/wiring.go`'s deps, plus a grep of `internal/boardengine/board.go`: to learn whether the board status write pushes synchronously. It does not.
