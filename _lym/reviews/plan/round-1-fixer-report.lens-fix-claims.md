```yaml
lens: fix-claims
outcome: reported
```

The fork checked whether the factual claims the fix added or rewrote hold against the code at HEAD.

Findings:

- **LOW, card 5, the added DoneWhen sentence ("…so a resume never drops an unawaited strand as finished and relaunches it").** The relaunch half is stated without its limits. The drop half holds: run.go:609-611 hands reed `OutputFiles` as `DoneWhen` only for an ungated spec, and `reedengine/lifecycle.go:196` drops a strand only when `DoneWhen` is non-empty and every path exists. But `planResumeLaunches` (`lifecycle.go:190-210`) relaunches a non-live strand only when it is not hidden and no pause covers it; under a pause it is held instead. A reader could take "relaunches it" to mean always. The fix is to add "unless a pause holds it" or to say "is treated like any other non-live strand".

These claims checked out:
- **Card 8:** the github.com host test and the new rows hold against card 1's `ParseOwnerRepo` change. Card 1 strips only `ssh://git@github.com/` and a non-empty-user `https://<user>@github.com/`. So `https://@github.com/o/r`, `ssh://git@github.com:22/o/r.git` and `git://github.com/o/r` are refused, and are on github.com by the host rule, so each halts. `ssh://git@gitlab.com/o/r.git` is another host, so it is a no.
- **Card 5, the wait.go restatements:** they sit at wait.go:73-74 (file header), above `classifyDeadlineExpiry` (wait.go:1497, function at 1503) and above `finishedDespiteMechanismFailure` (wait.go:1529, function at 1533). The `attach.go:532` comment is inside `dispositionCandidate` (attach.go:511). Attach never calls `Spec.validate`; only run.go:524 does. So the rewritten wording, "set-matches … against the caller's spec", is correct.
- **Card 5, "every Attach caller passes an awaited spec naming its output files":** not fully verified. The callers found are sheddriver, shedarbiter, shedadapters/bouncer, frictionengine and shedfake. Each attaches an existing agent spec, and none is unawaited, but I did not read each spec's `OutputFiles`.
- **Card 11, the `pr-review` default gate key:** `landingRowDeps` lower-cases the row name (`entries_simple.go:34`), so `PR-Review` gives `pr-review`.
- **Card 11, step 8:** "UpdateRecord on an absent record writes nothing" agrees with card 7 (`ErrNoRecord`, writes nothing).
- **Card 4, the gate-slot sentence:** PR-Review "runs its own `pr-review` key's call through `Verify`" holds. `verifyGate` reaches `verifytree.Verify` (`landingshed/verifygate.go:140`), and the wording matches its sibling sites (`PATTERN-gate-slot.md:61-63`).
- **Card 16:** `internal/hubissues/hubissues.go` is the only non-test file calling `.Issues.Edit(`, so the new allowlist breaks no current file.
- **Card 6:** the claim that a run-record commit fires a detached push is consistent with `fabricengine/commit.go:103-112`, which fires the async push of whatever landed.

Tool calls beyond context:
1. Card 1 and `githubclient/parseownerrepo.go`: card 1's new accepted shapes were not in context, and the card 8 rows depend on them.
2. wait.go function lines near 1440-1540 and the Attach callers outside shuttleengine: to place the restated comments and test the Attach-caller claim.
3. `reedengine/lifecycle.go` `planResumeLaunches` and the `fabricengine` detached-push lines: to test card 5's DoneWhen/relaunch claim and card 6's detached-push claim.
4. `fabricengine/commit.go:100-112`, `landingshed/verifygate.go` and `PATTERN-gate-slot.md:60-66`: to test the detached-push wording and card 4's `Verify` site sentence.
