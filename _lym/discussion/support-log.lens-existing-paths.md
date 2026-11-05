```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: I traced the code paths that already write or read the state each decision touches, and judged whether each decision survives them.

**Findings**

1. **BLOCKING — D4, D7, D8: a beam forked from a beam inherits its parent's tracked records.** The record has no beam slug, so the inner beam can pick up the outer beam's pull request.
   - Evidence: `internal/pawlengine/ready.go` says the ready record "names the beam it was written for, since a pair forked from the beam inherits the beam's records". `ReadyFor` and `RemoveReady` filter by slug for that reason.
   - D4 puts a `pullrequest.Record` under the same landing directory. Its fields are Number, URL, Base, Head (a branch name), GatedHead and ReviewEnded. None of them names the beam.
   - Effect on D8: the inner beam's Finalize finds a record "standing" and squash-merges the outer beam's pull request.
   - Effect on D7: the inner beam's dobby announces landings for a pull request that is not its own.
   - Effect on D3 refusal 6: it reads the outer beam's pull request.
   - The record needs to name its beam, or be read only when its Head is this beam's branch, the way `ReadyFor` filters. Removal needs the same filter.

2. **MEDIUM — D3 refusal 4 and D5: an inherited description passes the check.** A nested beam inherits `_lym/landing/summary.md` from its parent beam.
   - Today Beam-Describe rewrites that file at every landing, so the inherited copy never reaches Finalize. D1 drops that row and D5 says "nothing at landing rewrites `summary.md`".
   - The inherited file is valid, so refusal 4 passes. The inner beam then lands with the outer beam's title and body as its squash commit text.
   - Refusal 4 needs to check that the description was written for this beam, or `lym beam describe` must be required after Beam-Up.

3. **MEDIUM — D11: the policy alone may not re-step Beam-Await.** The interrupt policy is only reported. A row is re-stepped only where the arbiter or the driver rules act on it.
   - Evidence: `internal/arbiterstop/arbiterstop.go` describes the stops it may take as "a tapestry, darn or sampler stop … of kind interrupted under the reinvoke policy". Pawl is not in that list.
   - `NewBattenReaders` gives every row of a batten run the hand-back policy.
   - Adding Beam-Await to `loomshed.InterruptPolicies` may therefore still need a hand `lym loom resume` for #57.
   - D11 has to say who re-steps a pawl run (`internal/sheddriver/driverules.go` or the arbiter's recipe scope), and change that path too.

4. **MEDIUM — D4 and D8: Finalize writes and removes the tracked record without committing.**
   - Finalize's record calls (`openAtOnce` → `WriteRecord`, `dropRecordedPullRequest` → `RemoveRecord`, and D8's removal after the merge) were written for the scratch path `shedrun.PullRequestRecord`, which `internal/loomcli/landingdeps.go` wires today. None of them commits.
   - Moved onto D4's tracked path, they leave the weft dirty or out of step until something else commits. D4 says that "Finalize's landing remove[s] and commit[s] it", but D8's merge-at-once path writes the record and never commits it.
   - Another machine adopting the run mid-landing can then read a stale record.
   - D8 needs to say that every record write and removal Finalize makes on the tracked path is committed. Otherwise Finalize should keep its scratch record and only read the tracked one.

5. **MEDIUM — D13: two loose ends in the move.**
   - The PR-Review wait watches the scratch record path (`watchFileChanges(shedrun.PullRequestRecord…)` in `landingdeps`). D13 copies that record into D4's record but does not say the scratch file is removed. A task-shaped reader of the scratch path could act on it later. The record should say the scratch file is removed after the copy.
   - A copied record does not set D7's last-announced head. The fallback D7 names, "the record's recorded head", is not a field of `pullrequest.Record`; the closest is GatedHead, a SHA that PR-Review set. The record should name the field the fallback reads.

6. **LOW — D2 and D12: board status now names Finalize while shed is still at Beam-Await.** Between the accept and Done, the board status says Finalize but shed's current producer is still Beam-Await.
   - D12 lets `lym beam start` through for a beam "held by its own pawl run at Finalize". That opens the window in which Beam-Await has stopped the dobby and is re-checking.
   - If the re-check fails, `refuse` restarts the dobby while D12's start has already brought the orch up. That is harmless, but D12's "dobby never started on a held beam" is broken by `refuse`'s restart right after the hold lifts. That breakage is correct, because the hold is gone.

7. **LOW — D8: Finalize's merge-in can stall on the pull request's checks.**
   - If Finalize's merge-in brings parent commits, the beam head moves when it is pushed and the pull request's required checks run again.
   - `mergeOnGitHub` retries a pending refusal for only `prMergeRetryWindow` (one minute) before it halts.
   - D3's bound accepts this, but a beam whose parent moved after ready will usually halt at Finalize. D8 should say so, or wait for the checks under the parent lock.

8. **LOW — D7 next to `notify_on_land`.** A member that carries `notify_on_land` while D4's record stands gets two notices: the existing one in `internal/dobbyengine/landnotice.go` and D7's. Harmless, and the record is silent about it.

**Paths traced**

| Decision | Path | Result |
|---|---|---|
| D1 | `loomshed.OperatorWaitRows` lists Beam-Await and PR-Review | Survives; the PR-Review entry stays for tasks. |
| D1 | `landingdeps.go`'s Finalize land check over members, documented as pawl-only | Survives. |
| D2 | `boardengine.RunHolds` readers: orchcli ownerpause → `beamUpRefusal`, adoptcli, battencli beamfork, refusal, dobbyreads, loomcli peers, hubgeom | Survive; a hold now starts at the Finalize status. Window noted in finding 6. |
| D3 | dobby land step (`dobbyengine/land.go` `Passes`) | Survives; ready now needs nothing open. |
| D3 | `lym beam land` | Survives. |
| D4, D8 | pull-request record wiring: `shedrun.PullRequestRecord` in `landingdeps`, PR-Review watch, `openAtOnce` and `drop` writes | Breaks on commit and on inheritance (findings 1 and 4). |
| D5 | Beam-Describe's removal against `summary.md` inheritance | Breaks (finding 2). |
| D10 | `landingdeps.readReopenRequest`, `hubgeom.PawlPaths` ReopenRequest path, `beamcli` deps and doc, envkit | All go with D10. The record is silent on the `landingdeps` and `hubgeom` seams, which is fine at plan level. |
| D11 | `shedverbs` status and loopwait report the policy; arbiterstop eligibility | Survives as a report only; breaks for auto re-step (finding 3). |
| D12 | `orchcli.BeamUp` → `beamUpRefusal` held check | Survives with D12's change; `refuse` restart noted in finding 6. |
| D13 | `shedengine` step lookup, PR-Review scratch record, D7 marker | Partial (finding 5). |

**Tool calls**

1. Grepped `dobbygeom`, `landingdeps`, `operatorwait`, `prcontinue`, `peers` and `dobbyreads` for the removed row names and reopen readers. My context had only the subagent's package list.
2. Grepped the pull-request record wiring (`RecordPath`, `shedrun.PullRequestRecord`) and Finalize's dirty checks. My context did not show where the record path is wired.
3. Grepped `orchcli` `refusal.go` and `ownerpause.go` held logic, and `pawlengine/ready.go` for the inheritance note. Needed to confirm D12's path and that records are inherited.
4. Grepped readers of `InterruptPolicyFor`. Needed to know who acts on D11's policy.
5. Grepped `arbiterstop` interrupt eligibility. Needed to confirm whether pawl stops are eligible for a re-step.
