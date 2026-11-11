---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:32
    summary: "D1's Webster-row refusal is bypassable by `lym loom goto --to <Webster row>` after a fold, and the Bound does not say what that lets through"
    fix: "In D1's Bound, state that the Webster refusal reads the row the run stands at when the fold is made, so a fold made after the last Webster row followed by `lym loom goto --to` a Webster row (the way forward a Drift-Verify Stuck names as a fix through Webster) starts a fresh Master whose prompt renders only Added after Discussion entries, and the folded call is then absent from it. Add the same line to Open risks. In D6, add a fourth point to the parent directive's list: before sending a run back to a Webster row after a fold, record each folded call again with `lym loom decision add`. Add one Review acceptance bullet that the parent directive says so."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:82
    summary: "D2's cut keeps only versions committed at or before the trailer commit, but names no mechanism that orders a version against the trailer commit, and an empty trailer commit is no file version"
    fix: "In D2, state how the order is decided. Recommended: the new file-history seam walks the records branch first-parent, like the trailer-list seam, and returns each version with its SHA and its first-parent position; the trailer-list seam returns each commit's first-parent position too; `internal/loomcli` keeps a version when its position is at or before the trailer commit's. Say that committer time is not the order, because commits in the same second tie and the existing path walk follows committer time. Add an acceptance bullet for the empty-trailer variant: HEAD already holds X from a sweep, replace lands an empty trailer commit, an add makes Y, a writer restores X and a sweep commits it, and the check still reports added-decision-dropped for Y."
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:143
    summary: "D4's live-weft refusal names no fabric seam; the test it copies is unexported in fabricengine, reads origin, and boardcli may not run git"
    fix: "In D4, name the seam. Add to the Constraints line for PATTERN-fabric-boundary and PATTERN-gitrepo-client-boundary that fabricengine gains one general exported function reporting whether a slug's weft branch is live by the test Topology.Add adopts by, built on the code Add already uses (the local branch check and the origin tip and archive-tag probe), so Add and the upsert cannot disagree. State what the upsert does when the origin probe fails: refuse, with a way forward to re-run once origin answers. Add that refusal to the D4 refusal acceptance bullet."
  - id: F4
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:108
    summary: "D3's second copy case reaches a beam project-record run whose entry carries a stored record, which the code today never copies, and D3 does not decide it"
    fix: "In D3, say that the second case applies to a run whose record is the discussion record and excludes a run whose record is a beam's project record (a recipe with a Beam-Await row, such as pawl), because that record is seeded from the entry's body while a family project's stored record is the family filing's own. Add the exclusion to the D3 acceptance criteria as a bullet that `add` on a beam run leaves the project entry's stored record unchanged."
  - id: F5
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md:211
    summary: "Open risks omit the effect D4 leaves when a raddle start adopts a live weft: the run reads the motif run's committed record, not the stored one"
    fix: "Add one Open risks line: D4 refuses `--record-file` while the weft is live, but `lym board promote` is not refused, so a raddle run started in that window adopts the weft and keeps the motif run's committed record, which Record-Write never overwrites; the board's stored record does not reach it, and D3's later copy writes that adopted record over the stored one. State that the window closes when the pair's teardown finishes, and that D5's warning does not cover it."
---

### [MEDIUM] D1's Webster-row refusal is bypassable by a goto after a fold

**Location:** `_lym/discussion/decision-record.md:32`, with the Bound at line 66 and D6 at line 179.

**Issue:** D1 refuses a replace on a run that stands at a row past its plan's approval and not past its last Webster row.
Its reason is correct: `webster.runlevel` reads `discussionparser.AddedDecisionTexts` at each fresh Master spawn, and `RenderMasterPrompt` renders only those entries into `{{.parent_decisions}}`.
The guard reads the run's row at the moment of the fold, not later.
`shedverbs` offers `lym loom goto --to <row>`, and the tapestry recipe's own header names "a fix through Webster" as the way forward of a Drift-Verify Stuck.
So a fold at Code-Bouncer or any later row, then a goto to Webster, spawns a Master that never sees the folded call.
The Bound lists what the caller guard, the lock and the checks hold, and omits this route.
The rubric's attack-surface question asks that a guard that is bypassable state what it lets through and what bounds it.

### [MEDIUM] D2's cut has no stated ordering mechanism

**Location:** `_lym/discussion/decision-record.md:82`, lines 82 to 95.

**Issue:** The cut is "the newest version whose bytes hash to that trailer's sha256 and that was committed at or before the trailer commit".
That predicate compares a version's commit with the trailer commit.
Two facts in the code make the comparison undefined as written.
The replace can land an empty trailer commit (D1), and a path-limited history walk never lists a commit that left the path alone, so the trailer commit is then not among the versions.
`gitrepo.Repo.PathRevisions`, which `CommittedAnchoredFileHistory` uses, orders by committer time and does not follow first parents, while the trailer-list seam D2 adds walks first parents.
Commits in the same second tie, and a replace followed at once by an add is the ordinary case.
An implementer who orders by time gets a wrong cut exactly in the sequence the acceptance bullet at line 238 pins.
The rubric treats a rule that leaves the downstream a fact to find out as a finding.
The fix records the mechanism.

### [MEDIUM] D4's live-weft refusal has no seam

**Location:** `_lym/discussion/decision-record.md:143`; Constraints at line 199.

**Issue:** D4 says the refusal uses "the test `fabricengine.Topology.Add` adopts by".
In `internal/fabricengine`, that test is `weftBranchExists` and `probeWeftLeftover`, both unexported, and the probe reads origin through `remoteHeadTip`.
`internal/boardcli` imports `fabricengine` but may not run git, and `PATTERN-fabric-boundary` allows a module only a general seam.
The Constraints bullet for the fabric boundary names the two D2 seams and `CommitEmptyWithTrailers`, and none for D4.
The plan writer must invent the seam and the failure behavior (an origin that does not answer), which D4's acceptance bullet does not cover.

### [LOW] D3's second case silently covers beam project-record runs

**Location:** `_lym/discussion/decision-record.md:108`

**Issue:** `internal/loomcli/decision.go` documents that a run whose record is a beam's project record copies nothing, and the pawl recipe has a Beam-Await row and no Discussion approval row.
D3's second case is "a recipe with no approval row and a board entry carrying a stored record".
A family project's entry carries one (D3 and D4 both say the filing writes the project's own record).
So `add` on a pawl beam run for that project would copy the body-seeded project record over it.
D3 decides nothing about this.

### [LOW] Open risks omit the adopted-weft effect

**Location:** `_lym/discussion/decision-record.md:211`

**Issue:** D4 decides that the refusal counts a live weft, in-flight teardown included, and says why the record would not reach an adopting raddle run.
Open risks, which the focus directive asked to carry the true effect, say nothing of it.
The residual effect is that `lym board promote` stays admitted, a raddle start in the window adopts the weft, and that run keeps the motif run's record, so neither D4 nor D5 reaches it.

## Settled focus questions

- D2 against `PATTERN-fabric-boundary`: no defect. The two seams (every first-parent commit carrying a trailer key with its value, and a file history with SHAs) are general, and `Lym-Record-Replaced`, its `by path sha256` parsing, the path check and the cut stay in `internal/loomcli`, as the constraint line says. `CommitEmptyWithTrailers` is a general commit seam too. I ran `git commit --allow-empty -m msg -- <path>` with the path unchanged and another path staged under git 2.53.0: it landed an empty commit and left the other path staged, as D1 claims. The only boundary gap is F3, on D4.
- D2's cut against the sequence replace writes X, add makes Y, a writer deletes Y leaving X, a sweep commits X: no defect. The cut is the newest matching version at or before the trailer commit, so the sweep commit after it is not the cut, Y's version stays in view, and the check reports `added-decision-dropped`. The acceptance bullet at line 238 pins it. F2 covers the empty-trailer variant and the ordering mechanism.
- D4's adopted-weft case: the record decides it. `Topology.Add` adopts a live weft and keeps its run records, and `loomshed/recordwrite.go` never overwrites an existing record file. D4 refuses while the weft is live and counts the detached teardown window as live. Open risks omit the effect (F5), and the test has no seam (F3).
- D3's stored-record copy for a run re-entered into its Discussion segment: no defect. `loomrecipe.PastDiscussionApproval` returns past only when the current row's index is after the approval segment's, so a run sent back before it copies nothing, and the second case needs a recipe with no approval row. F4 is a separate reach of the second case.
- D1's refusal and D6's fold against Master's view: the Master half holds at the row (`AddedDecisionTexts` is read per fresh spawn), and the gap is F1. A Record-Write record cannot meet it: only the tapestry recipe has an Implementer (Webster) row, and tapestry has no Record-Write row. The darn stencil's rule that added entries win over the fenced Goal body is why D6 has the fold rewrite the body, and it is correct.
