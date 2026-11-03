```yaml
lens: code-affected-set
outcome: reported
```

**Task:** find what the diff's changes force outside the lines it touches (code-affected-set lens).

**Per-change set**

1. `planPass` / `drivable` / `drivePasses` now admit a paused run with `ReachedParent`.
   - Found: `runNotices` (`runnotice.go:167`, running only), `view.go` (the `Paused` field describes tasks, not runs), `heartbeat`/`runLabel`, and `internal/dobbyengine/doc.go`.
   - Covered: `runnotice.go`, `view.go` and `heartbeat`/`runLabel` need no change, and the reworded `doc.go:71-72` bullet matches.
   - Not covered: `doc.go:70` still says "A run gets a pass only while batten's state is running". That is now false.
   - Not covered: `contracts/orchrole/orch-role.md:46` opens "A halted or paused batten run waits for you: dobby does not resume it." The added exception sentence follows it, so the pair is consistent.
2. `RunView.ReachedParent`, filled by `battencli.DobbyRuns` for paused runs only.
   - Found: the `beamcli/doc.go:135` bullet (updated), the callers `beamcli/dobbydeps.go:125` and `DobbyViewReader.Read`, and the tests in `beamcli/dobbypairanchor_test.go`.
   - Covered: the bullet and the callers.
   - Not covered: no test reads the new field. `dobbypairanchor_test.go` and the battencli dobby tests never set a paused status, and `TestPlanPass` feeds `ReachedParent` by hand. That belongs to the code-tests lens, so I'm not filing it here.
3. `lym batten pause` stops holding a run that has reached its parent.
   - Found: the orch-role item 3 sentence (added). The Pause help in `battencli/cli.go:261-278` is silent.
   - Found: `fabriccli/anchor.go:318` tells the operator to quiesce live batten runs with `lym batten pause`.
   - Found: `pairteardown/claim.go:102` tells the operator to "turn a running run into an abandon with `lym batten pause`".
   - Found: the `pairteardown/doc.go:78` warn text.
   - Covered: the orch-role sentence.
   - Not covered: the three operator-facing pause pointers are not updated.
4. Commit message and `Pattern-Departure:` trailers.
   - The commit has no trailer. No diff line touches an entry that forbids the change, so none is needed.
5. PATTERN.md and `pattern/` declared changes.
   - The directive's declared-changes list is empty, so there is nothing to land.
   - The diff touches no `PATTERN.md` or `pattern/` file.
   - Not touched: `pattern/PATTERN-owner-machine.md`'s Passes section says a "full pass" is a resume and every hand pass is one. It does not say dobby gives none to a paused run, so it stays true.

**Findings**

1. LOW, `consistency`, `internal/dobbyengine/doc.go:70`.
   - Summary: the bullet still says "A run gets a pass only while batten's state is running", but `drivable` now admits a paused run with `ReachedParent`.
   - The two following lines (71-72) contradict it.
   - Evidence: `doc.go:70-72`, `passes.go:62-64`.
2. LOW, `consistency`, `internal/dobbyengine/doc.go:136`.
   - Summary: it says run notices go to "every listed run whose batten state is running". That is still true, because the new paused-and-reached run gets no run notice.
   - Verdict: not a defect. I record it only as checked, so it is not a finding.
3. LOW, `consistency`, `internal/fabriccli/anchor.go:318` and `internal/pairteardown/claim.go:102`.
   - Summary: both messages advise `lym batten pause <slug>` as the way to quiesce or abandon a run.
   - After the diff, a batten-paused run whose work reached its parent is re-driven by dobby through Land and Worktree-Teardown. The advice no longer holds for that run, and the orch role now names `lym tender pause` instead.
   - Evidence: `anchor.go:318`, `claim.go:102`, `orch-role.md:54`.
4. NIT, `consistency`, `internal/battencli/cli.go:261-278`.
   - Summary: the `lym batten pause` help says nothing about dobby finishing a reached-parent run that is paused.
   - The orch role carries the caveat. The operator-facing help does not.

I did not file Finding 2.

**Tool calls**
1. A grep of `view.go` for pause/paused, of docs and Go for "no pass"/"dobby does not resume" phrasing, and of `battencli/cli.go` for pause. My context lacked these consumers.
2. A read of `battencli/cli.go:258-285` and a grep of `batten pause` across md, go and yaml. My context lacked the help text and the other pointers to `lym batten pause`.
3. A grep of docs, contracts, `doc.go` files and `pattern/` for "state is running" and dobby/paused phrasing, plus the commit's trailers. My context lacked a check for other stale prose and for a `Pattern-Departure:` trailer.
