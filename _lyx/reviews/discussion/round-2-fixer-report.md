# Fixer report — discussion, round 2

## Changes

- F1, fixed: the decision record's Open risks for a writer that committed, or left uncommitted edits, no longer name `lyx loom goto --to Merge-In`.
  Each states that `gotoAdmitted` in `internal/shedengine/goto.go` admits only Preflight and Darn-Write from a Darn-Write halt, and that goto's refusal says it never skips a review row.
  The record names no in-run way forward that keeps the writer's work.
  It names the parent's recovery instead: drop the writer's work, meaning reset the task branch to the record's `head` and discard every uncommitted change, then `lyx loom goto --to Darn-Write` and `lyx loom resume`, where D3's entry check reroutes to Merge-In.
  The fix takes the branch the `fix` key offers for a case with no route through the perch, so D3 gains no new sentence.
- F2, fixed as proposed: D1's opaque bullet now states the contract of `parseVerifyFailures`, a log that yields only the opaque identity.
  D1's rationale reads "every uncertain case the parser reports".
  An Open risks bullet names the residual: an unparsed failure beside a parsed failure outside the impacted set, bounded by D2's one Merge-In re-entry and the hand-back.
  The opaque case of the third acceptance criterion and support-log Q3 now use the same wording.
- F3, fixed as proposed: the `PATTERN-fabric-git` constraint names fabricengine's parent-tips reader for D3's tip read.
  For the status read it names the dirty-paths reader Merge-In's clean check uses, a read-only `status --porcelain` the pattern exempts.
- Support log: a Round 2 entry under Review rounds.

## Self-check

- fix-consistency, MEDIUM: discarding uncommitted edits alone cannot restore D3's HEAD condition, because Darn-Write's landing commit moves HEAD.
  Disputed, premise false: `Commit` runs `CommitAnchoredPaths`, which stages and commits through `gitrepo.New(weftPath)` (`internal/fabricengine/commitweftpaths.go:82`).
  It commits to the weft records checkout, never to the task branch, and the claims fork reached the same reading.
- fix-consistency, LOW: support-log Q3 still described the old opaque rule. Fixed as proposed.
- fix-proposal, MEDIUM: the same landing-commit premise as the consistency MEDIUM. Merged with it and disputed, premise false, on the same evidence.
- fix-proposal, F2 and F3: no case found.
- fix-claims: no findings.
- fix-way-forward, LOW: a writer that both committed and left untracked or uncommitted files fails D3's clean-tree condition after a reset alone.
  Fixed with my own change: the committed-writer recovery also discards every uncommitted change, untracked files included.
  The two Open risks bullets stay separate rather than merged into one rule.

## Disputed

- Self-check MEDIUM, the landing commit moving HEAD, raised by fix-consistency and fix-proposal.
  Ground: premise false. The landing directory is committed to the weft checkout (`internal/fabricengine/commitweftpaths.go:82`, `commitRecordsPathsHeld`), not to the task branch D3's HEAD condition reads.

## Deferred

None.
