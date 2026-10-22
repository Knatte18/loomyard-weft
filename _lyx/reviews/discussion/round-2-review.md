---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:153-154
    summary: Open risks 2 and 3 name lyx loom goto --to Merge-In as the parent's way forward from a Darn-Write halt, a forward goto that gotoAdmitted refuses and that would ship writer commits past the Darn-Review perch and Describe if admitted
    fix: In decision-record.md's Open risks, rewrite the second and third bullets so neither names `lyx loom goto --to Merge-In` from a Darn-Write halt. State the fact with its evidence, that `gotoAdmitted` in internal/shedengine/goto.go admits only rows up to the halted row, so from a Darn-Write halt only Preflight and Darn-Write are admitted, and goto's refusal says it never skips a review row. Then give each residual a way forward that is both admitted and keeps writer commits behind the perch. If none exists for a writer that already committed, add one sentence to D3 giving that case a route through the perch, or state in the risk that the run halts for the parent with no in-run way forward, and name the recovery the parent actually has.
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:51
    summary: D1's opaque-failure rule cannot fire beside a parsed identity, because the verify parser emits its opaque identity only when it finds no test or package identity, so an unparsed failure next to a failing test outside the set classifies as not owned
    fix: In D1, replace the bullet on line 51 with the parser's real contract and its evidence, that `parseVerifyFailures` in internal/websterengine/verifyparse.go returns one opaque identity only when it found no test or package identity, so the bullet reads as a log whose only identity is opaque, or a record naming no failing test. Then add one Open risks bullet stating that an unparsed failure beside a parsed failure outside the impacted set classifies as not owned, bounded by D2's one Merge-In re-entry and the parent hand-back.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:139
    summary: The PATTERN-fabric-git constraint says D3's status read goes through existing fabric readers, while D3 itself names Merge-In's dirty-paths reader, which is verifytree's read-only git status and not a fabricengine reader
    fix: In decision-record.md's Constraints, replace the PATTERN-fabric-git bullet with one saying that D3's tip read goes through fabricengine's parent-tips reader, and that its status read reuses the dirty-paths reader Merge-In's clean check uses, a read-only `status --porcelain` that PATTERN-fabric-git exempts.
---

### [MEDIUM] Open risks name a refused goto that would also skip review

**Location:** `_lyx/discussion/decision-record.md:153-154`

**Issue:** The second and third Open risks cover a writer that committed, or left uncommitted edits, while the parent fixed the failure.
Each names `lyx loom goto --to Merge-In` as the parent's way forward, and the third adds having the edits committed first.
The run in both risks is halted at Darn-Write: the writer has looped through its gate budget, and Darn-Write has no `on_stuck`.
`gotoAdmitted` (`internal/shedengine/goto.go:288-337`) admits only rows up to and including the halted row, plus the halted entry's routed row, which a Stuck with no `on_stuck` does not have.
From Darn-Write, the recipe's second row, that leaves only Preflight and Darn-Write.
The refusal at `goto.go:161` says it outright: "goto only moves a run back, so it never skips a row's own work or a review or approval row".
So the named way forward is refused.
If it were admitted, it would send writer commits through Merge-In and Publish past the Darn-Review perch and Describe, the skip D3's bound (line 100) says the HEAD and clean-tree conditions exist to prevent.
That settles the first focus question: no automatic path ships unreviewed writer edits, but the record's own named recovery would, and it does not run.
The residual of a writer who committed is left with no in-run way forward: resuming re-spawns a writer whose gate cannot pass without the parent merge, and D3's HEAD condition keeps the reroute off.

### [LOW] D1's opaque rule assumes a parser behaviour that does not exist

**Location:** `_lyx/discussion/decision-record.md:51`

**Issue:** D1 counts as owned a log that "holds an opaque failure, one the parser could tie to no package".
`parseVerifyFailures` (`internal/websterengine/verifyparse.go:113-115`) emits its opaque identity only when it found no test or package identity at all.
A log holding a failure the parser cannot read beside a parsed failing test outside the impacted set therefore yields only the outside test.
It classifies as not owned, against D1's rationale that "every uncertain case counts as owned" (line 60).
On the focus's third question: a build, package-level or crashed-binary failure in an impacted package is parsed as a package identity (`verifyparse.go:102-104`) and stays owned, as D1's fourth bullet and the third acceptance criterion say, so that half of the claim holds.
Only the unparsed half rests on a false premise.
The cost is bounded: one Merge-In re-entry and a hand-back whose text wrongly places the failure outside the task.
Hence LOW.

### [LOW] Constraint names the wrong reader for D3's status read

**Location:** `_lyx/discussion/decision-record.md:139`

**Issue:** The Constraints bullet says "D3's tip and status reads go through the existing fabric readers, never raw git."
D3 (line 88) names the dirty-paths reader Merge-In's clean check uses, which is `verifytree.DirtyPaths`: `gitexec.Run` over `git status --porcelain -z` (`internal/verifytree/verify.go:256-262`), not a fabricengine reader.
`pattern/PATTERN-fabric-git.md` exempts read-only `status --porcelain`, so the design breaks no invariant.
The bullet does contradict D3, though, and a plan writer following the constraint would build a separate fabric status reader, the disagreement with Merge-In that D3's "so the two never disagree" rules out.

## Focus verdicts

- D3, worktree states and the skip-review reroute.
  The HEAD and clean-tree conditions hold for every automatic path.
  The run's status file is committed on the fabric sibling (`internal/statuscommit`), so it never dirties the task worktree or moves its HEAD.
  Darn-Write commits the landing directory after every session with a non-empty pointer, a gate-failed Stuck included (`internal/loomshed/darnwrite.go` `Call` doc), so a writer spawn moves HEAD and disables a later reroute, which errs safe.
  A gate-exhausted writer's uncommitted code edits fail the clean condition.
  Merge-In's "before the merge-in" clean check reads the same reader D3 does, so a rerouted run does not halt there.
  A Merge-In left Stuck without merging is re-entered from Darn-Write only by the parent's explicit goto, and shed's reroute cap bounds that loop.
  Not a defect, apart from F1's named recovery.
- D3, the parent tip under each push mode.
  Under push skip, Merge-In's `syncParent` reads `ReadParentTips(false)` and merges the local branch, the one tip D3 reads.
  Otherwise Merge-In fetches and fast-forwards the local branch to its upstream before merging it, so both the local branch and the tracking ref D3 reads become ancestors of HEAD.
  The ahead, diverged and dirty-pair cases are Stuck without merging, and D3's bound covers them.
  The ancestor claim holds in both modes. Not a defect.
- D1, a mixed log: see F2.
- D4, the file's writer and remover.
  D4 has each gated producer in `internal/shedadapters` told the path, writing through the shared reason helper on exhaustion and removing the file itself on a call whose gate outcome passed.
  The helper only writes, which matches its single exhausted-gate call site (`gateattempts.go`, called from `burlerfix.go` and `multillm.go`), and removal sits with the producer, which sees the passing outcome.
  Coherent. Not a defect.
