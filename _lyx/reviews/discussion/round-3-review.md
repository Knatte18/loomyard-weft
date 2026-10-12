---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D3 (way-forward paragraph), D5, D9, Open risks bullet 2, Acceptance criteria (run_settings refusal bullet)
    summary: The named way forward for raising a loom hard cap never reaches the running task worktree, so the D3 load refusal and the D5 circling-continue refusal leave an in-flight task with no working route to raise its cap
    fix: In D3, replace the two concrete routes with one route per layer the resolved key comes from, each run so it reaches the task worktree's own config, for example a value from the task's run_settings (the local layer) is raised with `lyx config loom --layer local --set <key>=<n>` (or `lyx config darn --layer local --set …`) in the task worktree, with `lyx board upsert` correcting the task's run_settings too so a later Seed-Child writes the same value; a value from the task worktree's own tracked layer is raised with `lyx config loom --set <key>=<n>` in the task worktree; a value from the hub default is raised with `lyx config loom --layer default --set <key>=<n>`, or `lyx config darn --set <key>=<n>` from the prime for darn; then `lyx loom resume` (D3) or `lyx loom circling continue <slug>` (D5). Make D5 and D9 point at D3's route instead of restating it, reword Open risks bullet 2's pointer to match, and change the acceptance criterion on a run_settings hard cap to name the local-layer route.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2 Bound (paragraph after step 5)
    summary: D2's Bound for a mis-tagged mechanical fix rests on an operator review at PR-Gate that does not happen by default, and claims a downstream review segment that Merge-Review does not have
    fix: In D2's Bound, state that on the Merge-Review and Darn-Review perches no later review segment reads the accepted artifact, and that the only later gate is PR-Gate, where the alignment agent approves and lands the task with no human read while `alignment_pr_approve` is true (the default), and the parent decides only when it is false. Keep the soft-cap window, the `automatic` flag and, for Merge-Review, the `## Open risks` carry-over as the stated bounds, and add this to Open risks bullet 1 if the residual risk is accepted.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md Acceptance criteria, bullets 2, 4, 5, 7 and 8
    summary: Several acceptance criteria state the rule's continue, circling or hard-cap outcome without the auto-accept qualifier their path needs, and contradict D8 and the criterion that a non-Merge auto-accept perch accepts a harvested round with a design-wrong entry open
    fix: Qualify each of these criteria the way the earned-CIRCLING criterion already is, on a perch without auto-accept, on a replay, or on a blocking_escalates round with an open BLOCKING entry. That covers the soft-cap automatic continue (bullet 2), the returning-key circling escalation (bullet 4), the one more round after a LOW or mechanical previous entry (bullet 5), the counted-degrade soft cap the rule settles (bullet 7), and the hard-cap escalation (bullet 8). Merge-Review defaults to `merge_auto_accept` true, so an unqualified criterion there would assert an outcome D8 rules out.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D3 (paragraph on measuring the hard cap by round number)
    summary: D3 says no round after round H runs, but a goto onto the segment's review row runs round H+1's review and fix before the Bouncer settles it, so the guarantee is overstated
    fix: In D3, replace "so no round after round H runs" with the guarantee the design delivers, that the Bouncer never continues a round numbered H or higher, so a round at or past H is accepted or escalated whatever produced it, a goto onto the segment's review row included.
---

### [BLOCKING] Hard-cap way forward never reaches the running task

**Location:** `_lyx/discussion/decision-record.md` D3 (way-forward paragraph), D5, D9, Open risks bullet 2, and the acceptance criterion on a run_settings hard cap.

**Issue:** D3 names `lyx board upsert` then `lyx batten run <slug>` for a hard cap from the task's `run_settings`, on the premise that Seed-Child rewrites the local layer "on every call".
Seed-Child runs only while it is batten's current row: `internal/battenshed/doc.go` says "A resume redoes the whole write", meaning a resume of that row, and `internal/battenshed/deps.go` calls a revisit of a Done Seed-Child row "normally unreachable, since Seed-Child runs once and never revisits a Done row".
Both refusals arise after Seed-Child is Done.
The D3 load refusal comes from the cross-layer hard-below-soft check at loom load, which `configreg.PrepareRunSettings` cannot catch, since it calls `configengine.ValidateLocal` on the local document alone.
The D5 refusal comes from a run awaiting at a review perch.
At that point batten sits at Run-Shed, and `lyx batten run` "resumes silently from the persisted current producer" (`internal/battencli/cli.go`), so the board edit never reaches the task's `_lyx` local layer.
`lyx batten goto` refuses a running run and goes back strictly before the current row only for an awaiting one, while the batten process waiting at Run-Shed holds the lock, so it gives no route either.
The hub route is also wrong for loom.
Loom is not `HubWide` (`internal/configreg/configreg.go`), so `lyx config loom --set` with no `--layer` writes the anchor's worktree layer (`selectedLayer` in `internal/configcli/hubwide.go`).
Run from the prime, that is the prime's tracked file, which an already forked task branch does not read.
Only `--layer default` writes the board file (`configengine.LayerFile`) that every worktree stacks.
Darn is `HubWide`, so `lyx config darn --set` from the prime is right.
The failure lands on D9's own advice, `{"loom": {"review_max_bounces": 2, "review_hard_max_bounces": 3}}`, so every small task that follows the advice gets a hard cap the named route cannot raise.
That breaks `PATTERN-refusal-way-forward`.
`lyx config` is not among the denies in `internal/shuttleengine/claudeengine/settings.go`, so a local-layer route in the task worktree satisfies `PATTERN-no-denied-recovery`.

### [MEDIUM] D2's Bound relies on a PR-Gate review that does not happen by default

**Location:** `_lyx/discussion/decision-record.md` D2 Bound.

**Issue:** For Darn-Review the Bound names "the operator's review at PR-Gate".
`alignment_pr_approve: true` is loom's template default (`internal/loomengine/template.yaml`), and `internal/landingshed/pralign.go` applies the alignment agent's approve at PR-Gate, which lands the task with no operator read.
The loom-side sentence, "The segments downstream review their own artifacts built on it", does not hold for Merge-Review, the last review segment before PR-Gate.
Merge-Review defaults to `merge_auto_accept: true` with `blocking_escalates`, and under D8 its BLOCKING rounds now go to the rule, so an unreviewed MEDIUM-or-BLOCKING fix there is the case this Bound covers.
The Bound therefore overstates what limits an unreviewed, mis-tagged design change on two perches, which is the attack-surface statement the rubric requires to be accurate.

### [MEDIUM] Acceptance criteria miss the auto-accept qualifier

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria, bullets 2, 4, 5, 7 and 8.

**Issue:** D8 keeps auto-accept on a harvest: a non-Merge auto-accept perch accepts at the soft cap whatever is open, and Merge-Review accepts unless an open BLOCKING entry routes it through `blocking_escalates`.
The earned-CIRCLING bullet carries this qualifier ("on a perch without auto-accept, on a replay, and on a `blocking_escalates` round").
These bullets state an unconditional continue, circling or hard-cap outcome without it:
- "runs one more round, recorded as an automatic continue";
- the returning-key circling escalation;
- "runs one more round below the hard cap";
- "the rule settles that round";
- "At the hard cap, a design-wrong entry escalates with cause `hard-cap`".
On Merge-Review, auto-accept by default, an open MEDIUM design-wrong entry at the soft cap is accepted by auto-accept, not continued.
That contradicts these bullets and agrees with the bullet "A non-Merge auto-accept perch still accepts a harvested round…".
The plan writer's tests would pin one side of that contradiction.

### [LOW] "No round after round H runs" is overstated

**Location:** `_lyx/discussion/decision-record.md` D3, the paragraph on measuring the hard cap by round number.

**Issue:** `internal/shedadapters/doc.go` describes the review row as resolving "the round from the highest complete round".
A goto onto the review row of a segment whose round H is escalated and undecided therefore spawns round H+1's review and fix.
The Bouncer then settles H+1 by the rule, so the hard cap still stops the segment, but round H+1 has run.
The rest of D2 and D3 holds: the Bouncer stops any round numbered H or higher by the round-number leg, after a goto included.
Only this sentence promises more than the design enforces, and a test written from it would assert something false.

## Focus directive verdicts

- Hard cap versus a goto: holds.
  `Bouncer.Call` clears a generation only on a settled round, so round numbers carry on after a goto, and D2's round-number leg opens the window below a reset soft cap.
  D8's harvest auto-accept fires only inside `budgetReached`, so after a goto it yields to the rule too.
  The acceptance criteria state the goto case.
  The one overstatement is F4.
- Way forward for a `run_settings` hard cap: it does not unblock the task (F1).
- Recorded continue at the hard cap: consistent.
  `circling continue` refuses unless the run is awaiting, and a pending round needs an escalation record (`internal/shedadapters/circling.go`), so the "continue always answers an escalation" premise holds.
  D2 moves both files aside and writes a fresh `hard-cap` record, the "keeps its record" sentence covers only a round with no recorded continue, and the D6 brief and the acceptance criterion agree.
- Mechanical-tag bound: the carry-over is wired for all four loom perches (`carry_over:` in `contracts/recipes/loom-recipe.yaml`, MEDIUM-or-worse plus unlabelled entries in `buildCarryOver`) and is absent for darn, as the Bound says.
  The Bound's stated limits for Darn-Review and Merge-Review are wrong (F2).
- Premises in D2 step 4 and the criteria: the earned-CIRCLING premise holds.
  Below the checkpoint no CIRCLING is earned, and step 4 says a returning key there runs to the hard cap.
  The auto-accept qualifiers are missing from several criteria (F3).
