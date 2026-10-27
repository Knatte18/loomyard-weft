```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each reviewer proposal the fixer carried out, using concrete cases where the proposed change goes wrong.

1. **MEDIUM**, Decision 2, the re-exec test: "when it holds this process's pid and this process's own start ticks (`proc.StartTime`, which an exec keeps), the start is a re-exec". The F5 proposal breaks the first re-exec after the deploy that ships this change.
   - The old image's `recordStart` wrote only `pid` and `start_time` (`internal/dobbyengine/state.go:17-19`), so the record has no start-ticks field.
   - The new image reads its own pid with empty ticks and so fails the test. It treats itself as a fresh start and waits only `lock.ProbeGrace`, which is the incident's own window.
   - If it is refused, it takes the refused-fresh path and posts nothing. Decision 3's notice is lost on exactly the deploy that matters.
   - PATTERN-tolerant-records (as the Constraints cite it) needs a rule for a record that lacks the field. A sound rule: an empty start-ticks value with a matching pid counts as a re-exec. The record and the criteria need that row.

2. **MEDIUM**, Decision 10: "calls `hubreconcile.EnsureOrWarn` ... so Decision 9's migration has run whenever the check reads `adopt`", plus the new criterion "A check on a hub whose build stamp is stale runs `finalize`'s tuned tiers". The claim fails when the reconcile fails.
   - `EnsureOrWarn` logs a warning and goes on (`internal/hubreconcile/hubreconcile.go:59-63`). A hub-lock timeout is one way it fails: `Ensure` returns `*LockTimeoutError`.
   - On that failure the check reads the template's `adopt: {tiers: [0]}`, goes green on tier 0, and posts a green end notice.
   - The warning reaches only the detached check's stderr, which is not in `suite.log`, `check.json` or any post.
   - So F3's premise, a green check that skipped the tiers that catch a machine-move break, still ships in that case.
   - A better fix: call `hubreconcile.Ensure` and end the check as failed with the reconcile's way forward. If `EnsureOrWarn` stays, state the bound and name the reconcile failure in the end notice.

3. **LOW**, Decision 3, the wiring-failure bullet: "`runDobby` also posts the failed-re-exec notice to the nearest live orch, naming the error". The record names no route for that post.
   - The dobby's `Notice` and `NearestOrch` seams are built inside `hubDobbyDeps` (`internal/beamcli/dobbydeps.go:120,157`), which is the call that failed.
   - The record says the log line goes "without the `Deps.Log` seam it failed to build", but gives the notice no equivalent.
   - The plan writer has to invent one. Either name it (for example, build the notice from `prime` alone, outside `hubDobbyDeps`) or state that it is built separately.

4. **LOW**, Decision 10, the race the F2 fix leaves open: "When `check.json` reads running and its process is alive ... `lym adopt` spawns no check" together with "A check that finds the lock held exits 0 without writing".
   - Suppose a second `lym adopt` runs between the first spawn and that child's taking the lock. It still sees the previous ended record, so it spawns.
   - Its envelope's `suite` then names a pid that exits at once without running anything. The operator gets a pid for a check that never ran.
   - The Bound sentence covers only `lym beam status` showing the previous record, not this envelope.

Tool calls beyond context:
- I grepped `internal/beamcli/dobbydeps.go` and `internal/dobbyengine/state.go` for the Notice/NearestOrch wiring and the state record's fields. I needed to know whether a notice route exists outside `hubDobbyDeps` and whether old records carry start ticks, and my context had only part of `dobbydeps.go` and none of `state.go`.
