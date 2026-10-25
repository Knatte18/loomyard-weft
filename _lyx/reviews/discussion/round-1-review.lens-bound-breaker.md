```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I tried to break every bound, guard and claim about existing code in the decision record.

**Claims attacked**

1. D1, a `capture` or `compare` command without `{baseline_dir}` is refused at load, and both are executable keys read only from the template and the board default layer: held.
2. D2, with the flag off `Drift-Baseline` and `Drift-Verify` are Skipped and spawn nothing: held.
   `Drift-Await` still holds Webster when the flag is turned off mid-capture: held.
3. D2 and D6 together, under `full` an acceptance is Stuck "so the parent rules", while the arbiter may settle a Drift halt with `lyx drift accept`: broken (F1).
4. D3, `lyx drift scope` is refused outside `Drift-Baseline` and on invalid combinations: held.
5. D3, after a dead capture `lyx loom goto Drift-Baseline` "captures again": broken in loom, because the goto also re-runs Plan-Write and the Plan perch (F2).
6. D4, a main capture runs only at the branch point: held.
   `_lyx` is a symlink into the weft, so a records commit never moves HEAD, and HEAD equals its merge base with `origin/main`.
7. D4, a released lock is how every reader tells a dead capture: broken on Windows after a timeout (F3).
8. D4, a new `runproc` child entry is needed to end a timed-out command, and on Windows the capture must keep its lock and slot: broken, the premise is false (F3).
9. D4, the capture "holds no lock an agent's own work waits on": broken (F4).
10. D4 and D5, the timeout runs on the monotonic clock and `Drift-Await` is bounded by `timeout_min` plus one minute from command start: broken across a machine sleep (F5).
11. D4, "the backup is the only refresh, reachable only before the first code change": held.
    The row guard, the branch-point check and the dirty-tree check all close it.
12. D5, after an adopt `awaited_at` lets the run pass: broken, regression is lost on the adopting machine (F6).
13. D5, `lyx loom goto Webster` runs "without regression": broken (F7).
14. D6, the `verify_max_runs` budget bounds one `Drift-Verify` session's compares and side captures: broken (F8).
15. D6, every attempt's handoff runs the compare last: held within the row.
16. D7, only the `Drift-Verify` agent, the arbiter or the operator can accept: held, by the row guard.
    What an acceptance lets through under `partial` has no bound: broken (F9).
17. D7, nothing refreshes a baseline after the first code change: held.
18. Constraint, no row between `Drift-Baseline` and `Drift-Await` builds: held for builds, but the plan gate holds a gate slot (F4).
19. Constraint, `pairteardown` ends the capture through `EndSparing`: held, `internal/pairteardown/teardown.go:576`.
20. Open risks, a capture that ran nothing fails: held, because the backup rename empties the directory first.

**Findings**

F1. MEDIUM, design. Location: D6 "Under scope `full`…", D2 arbiter sentence, Acceptance criterion "Under scope `full`, any acceptance…".
The `full`-scope guard either lets the arbiter bless drift or makes the halt's own way forward loop.
Counterexample: under `full` the agent accepts, and the row is Stuck.
The arbiter, which comes before the parent in the stop chain (`internal/shedarbiter/doc.go`, overview line 639), runs the way forward "`lyx drift accept` and resume".
On re-entry the compare passes and no session accepted in that attempt, so the row returns Done and the parent never rules.
If instead the rule reads the scope record's accepted list, as the acceptance criterion says ("any acceptance"), every resume is Stuck again and "accept and resume" can never succeed under `full`.
D7's record has no field saying who accepted, so D6's "session that accepted" cannot be implemented as written.

F2. MEDIUM, design. Location: D5 "names `lyx loom goto Drift-Baseline`, which captures again (D3)".
In loom `Drift-Baseline`'s `on_done` is Plan-Write, so the goto also re-spawns Plan-Write fresh.
Its fresh-spawn preparation `NewPlanDirRotator` archives the approved plan (`internal/loomshed/planwrite.go:96`), and the run then goes through the whole Plan perch again.
The record presents the goto as a way to recapture only.
It states neither this cost nor a goto target that recaptures without re-planning.

F3. MEDIUM, design. Location: D4 "through a new `runproc` entry … on Windows … keeps the lock and slot until the command exits", PATTERN changes, Scope bullet "A `runproc` entry".
The premise is false.
`proc.StartTied` with `Tied.Kill` already kills a child's whole tree on Linux (process group) and Windows (kill-on-close job object) (`internal/proc/tied.go:33-37`).
The gate runs exactly this on a timeout (`internal/gatecli/child.go:20-45`).
The command is an unregistered child of the capture process, so ending it this way departs from no PATTERN entry, and the `PATTERN-run-process-owner` change is unneeded.
The Windows rule as written breaks D4's own bound.
A command that hangs past its timeout holds a hub gate slot and the capture lock forever, since nothing ends it on Windows.
Then `lyx loom goto Drift-Baseline` finds the lock held, starts no capture, and `Drift-Await` is Stuck again on the `failed` record, in a loop.

F4. LOW, design. Location: D4 "holds no lock an agent's own work waits on", Constraint "the plan gate's type load is Go-only".
The plan gate's type load is a non-tier gate-slot holder (`internal/gateslot/doc.go:36-37`).
On a hub with one slot, Plan-Write's gate waits behind the whole capture, so the claim is false.
The record should say the plan gate can queue behind the capture.

F5. MEDIUM, design. Location: D4 monotonic-clock sentence, D5 "bounded by `capture.timeout_min` plus one minute from its command start".
`Drift-Await` runs in another process and can read the command start only as a wall-clock time from `capture.json`, so its bound runs on a different clock from the capture's own timeout.
Counterexample: the capture starts at 10:00 with a 60-minute timeout, and the machine sleeps from 10:30 to 12:00.
After wake, `Drift-Await` computes 10:00 + 61 minutes, already past, and halts Stuck while the capture is healthy, holds its lock, and runs on until about 11:30 by its own clock.

F6. MEDIUM, design. Location: D5 "`awaited_at` … after … an adopt on another machine, pass without a baseline check", D6 "A scope with no passed capture record halts Stuck".
On an adopt, the old machine's owner pause ends the run's registered processes, the capture included (`tendercli` doc line 30, `EndOwnedElsewhere`).
`.lyx/drift/`, which holds the capture record and the baseline, is never-tracked, so the adopting machine has neither.
With `awaited_at` set, `Drift-Await` passes, and `Drift-Verify` then always halts Stuck naming the way to skip regression.
No recapture is possible, since code has changed.
The record presents the adopt path as working but states neither that regression ends at an adopt nor any carry of the baseline.

F7. LOW, consistency. Location: D5 "`lyx loom goto Webster` to run without regression", D6 "A scope with no passed capture record halts Stuck".
After a failed or dead capture the goto does not run the task without regression.
The scope record still says `partial` or `full`, so `Drift-Verify` halts a second time.
In the HEAD-moved case the capture had passed, so the run is in fact compared.
The way-forward text and D6 disagree on what the goto means.

F8. MEDIUM, design (attack surface). Location: D6 "`lyx drift compare` and `lyx drift capture --side`, whose runs count against `drift.verify_max_runs`".
Unlike `scope` and `accept`, no Decision restricts these verbs to the `Drift-Verify` row or states when the Go-written counter resets.
Counterexample: Webster's implementer, for which the record keeps `lyx drift` verbs allowed, runs `lyx drift compare` repeatedly during implementation.
Each run holds a gate slot, and either drains the later `Drift-Verify` budget or, if the counter is per session, runs unbudgeted.
A resume after a `Drift-Verify` halt spawns a new session, and the record does not say whether that session gets a fresh budget.

F9. MEDIUM, design (attack surface). Location: D7 `lyx drift accept`, under scope `partial`.
D7 states the row guard but neither what an acceptance can let through nor what bounds its count under `partial`.
Counterexample: the agent accepts every failing test.
The skip file then holds them all, the compare exits 0 on the rest (or on zero tests, per the Models open risk), and the row returns Done with no review and no Stuck.
The only bound is the agent's judgement, and that is stated only in the support log (bound-breaker 2), which Plan-Write never reads.

Tool calls:
1. `ls _lyx`, `git log`, merge-base and `internal/proc` grep: to check the `_lyx`-symlink and branch-point claim and whether a detach exists, neither of which the manifest held.
2. `tendercli/deps.go` function index and `EndOwnedElsewhere` callers: to find which verbs end a run's processes beyond the `End` sites read.
3. Grep for gate timeout and tree-kill sites: to test D4's claim that no route ends one child's tree.
4. `internal/proc/tied.go` index and `gatecli/child.go`: to confirm the existing tree kill on both OSes.
