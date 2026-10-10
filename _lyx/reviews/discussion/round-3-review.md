---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (D2 Decision and Bound)
    summary: Under zsh EXTENDED_GLOB an unquoted mid-word # is a glob quantifier, so a word like x#--output=f is admitted as a plain positional before -- yet expands to a planted file named --output=f; D2 neither rejects # nor records the residual
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D6 cases 2 and the dead-fork acceptance rows)
    summary: No code path makes a terminal dead fork record, since record-batch yields done, stuck or failed and only a recovery record is classified dead, so D6's dead-fork case and its three acceptance rows rest on an unreachable state
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md (D6 exhausted case; Scope Out on the Master stencil's rungs)
    summary: D6 turns begin-batch's stuck or failed remedy at the cap into an exhausted text naming reset --to batch-start, which Master's begin-batch rung does not list and its no-other-reset rule forbids, and D6 leaves open whether begin-batch carries recovery_exhausted as an envelope flag
---

### [BLOCKING] D2's glob list misses zsh's `#` quantifier, so a planted flag-named file still acts before `--`

**Location:** `_lyx/discussion/decision-record.md`, D2 Decision ("An unquoted `^` starting a word counts as a glob character") and Bound ("the classifier assumes bash or zsh, with any glob option set").

**Issue:** D2 rejects an unquoted word holding `*`, `?`, `[`, or a leading `^` before `--`.
It also keeps the rule that a dequoted word starting with `-` must match a listed flag exactly.
Its rationale is that a planted name spelling a flag can't act anywhere a glob is admitted.
Under zsh with `EXTENDED_GLOB`, `x#` means zero or more `x`, and `x##` means one or more.
So `x#--output=f` matches a file named `--output=f` (zero `x`s, then the literal tail).
The dequoted word starts with `x`, not `-`, so D2's flag rule never sees it.
`#` is not in D2's glob set, so the word passes as a positional of `git log` before `--`.
In a worktree holding a planted `--output=f`, `git log x#--output=f` is admitted and then runs as `git log --output=f`, which writes a file.
That is the exact attack D2's rationale and the support log's rejected alternative ("a planted file name spelling a writing flag would act before `--`") say the design closes.
The Bound claims "any glob option set", so the residual is neither rejected nor recorded.
The same reasoning covers `~` as zsh's exclusion operator only partly: `a~b` matches the literal left side alone, so it adds no new name.
Only `#` lets a pattern match a name whose first character differs from the word's.
The current classifier (`internal/fabricengine/refscanner.go:264`, `plainWord`) has no `#` handling, so nothing downstream catches it.

**Fix:** In D2, count an unquoted `#` anywhere in a word as a glob character under the same rule as `*`, `?`, `[` and a leading `^`.
That admits it only after `--` in `log`, `show`, `diff` and `status`, and rejects it in a `lyx` segment.
Add a rejected row such as `git log x#--output=f` under Acceptance criteria.
Alternatively, narrow the Bound to exclude `EXTENDED_GLOB`'s `#` and record the residual with its bound.
The support log already rejected narrowing for `^`, so rejecting `#` is the consistent choice.

### [MEDIUM] D6 builds cases and rows on a dead fork record no code path produces

**Location:** `_lyx/discussion/decision-record.md`, D6 Decision (second case, "a dead fork record names `lyx webster recover-batch NN`") and Rationale ("false for a dead fork batch that has had no recovery"); Acceptance criteria rows "A dead fork record below the cap…", "A dead fork record at the cap with no unrendered amendment…", "A dead fork record at the cap with an unrendered amendment…"; support log Interview item 7.

**Issue:** A fork record's terminal status comes from `RecordBatch`, which sets `distill(report)`'s status (`internal/websterengine/recordbatch.go:552-553`).
`distill` yields only done or stuck (`internal/websterengine/digest.go:44-52`), and `failBatch` yields failed (`batchfail.go:135-136`).
`DigestStatusDead` is produced only by `Classify` (`classify.go:44-52`), which recover-batch's await loop calls on the recovery record (`recoverbatch.go:667-668`, `Kind: "recovery"` at line 357).
The only other `Kind` assignment is begin-batch's fresh, non-terminal fork record (`beginbatch.go:542`).
The board entry's item 5 names a "terminal-dead refusal" that predates the second recovery, which is a dead recovery's text.
So the premise behind D6's dead-fork case and the support log's item 7 is false.
D6 then specifies, and asks the plan writer to pin with three rows, a branch that only a hand-built `state.json` reaches.
The plan writer can't tell whether a dead fork record is a real state to handle or a test-only fixture.

**Fix:** Drop the dead-fork case from D6 and its three acceptance rows, and correct support log item 7's premise.
Keep the count-before-kind ordering, since a stuck or failed fork record does carry the batch's recoveries across a re-begin (`beginbatch.go:524-545`).
If the author keeps the case defensively, D6 must say that no current path produces the state and why the case is kept.

### [MEDIUM] D6's exhausted remedy for a stuck or failed record names a route Merriam's begin-batch rung does not cover

**Location:** `_lyx/discussion/decision-record.md`, D6 first case ("a record of any kind and any terminal status but done, whose recoveries are at the cap… names… `1) lyx webster reset --to batch-start --batch NN; 2) lyx webster recover-batch NN`") and Rationale ("begin-batch and Merriam never disagree on one state"); Scope Out ("the Master stencil's rungs").

**Issue:** Merriam's begin-batch rung (`contracts/stencils/webster/webster-template-master.md`, "begin-batch <NN> refuses because the batch already has a report") tells it to "follow exactly that remedy".
It lists the remedies as `record-batch`, `recover-batch` or beginning the next batch.
Its only stop clause is "A dead batch with `recovery_retry: false` ends the run".
The same stencil says "Run `reset --to report-head` for a batch at most once, and no other reset".
D6 changes a stuck or failed record at the cap from `recover-batch NN` to an exhausted text whose steps are `reset --to batch-start` then `recover-batch`.
Merriam then has no rung for it: following the remedy breaks the no-other-reset rule, and the dead-batch stop clause does not cover stuck or failed.
recover-batch's own exhausted refusal reaches a dedicated rung through the `recovery_exhausted` envelope flag (`internal/webstercli/recoverbatch.go:196-197`).
D6 only says begin-batch "names… the `recovery_exhausted` state".
It leaves open whether begin-batch's refusal gains that flag, and Scope lists no begin-batch CLI change.
D6's claim that begin-batch and Merriam never disagree is argued only for a dead recovery, not for this case.

**Fix:** Decide how Merriam reads begin-batch's exhausted case without a rung change.
One option is that begin-batch's refusal carries `recovery_exhausted: true`, with D6 stating that Merriam stops stuck on it and the reset steps are the operator's.
That would add `internal/webstercli`'s begin-batch verb to Scope.
Otherwise, state why the Master rung needs no change and record the residual.
Either way, D6's Merriam-consistency sentence must cover the stuck and failed exhausted case.

## Verdicts on this round's focus

- D6, dead recovery that committed nothing with an unrendered amendment: not a defect.
  `RecoveryRetry` is false both below and at the cap (`recoverbatch.go:383`: kind recovery, dead, below cap, then the committed check), so case 1 is skipped for the amendment and case 4 ("end the run stuck") holds in both.
  That matches Merriam's `recovery_retry: false` stop, and ending stuck names no route recover-batch refuses, even though its gate (`recoverbatch.go:440`) would admit the amendment.
  The rule is one rule, and the support log records the rejected alternative.
- D6, default branch for stuck and failed: holds.
  Below the cap it names `recover-batch NN`, which the gate admits.
  At the cap with an unrendered amendment, case 1 is skipped and `recover-batch NN` is named, which the gate admits.
  At the cap without one, case 1 names exhausted.
  Its Merriam-side gap is F3.
- D6 rows, dead fork record at the cap with an unrendered amendment: the row exists ("names `recover-batch NN` and never exhausted").
  The state it pins is unreachable (F2).
- D2 Bound under zsh: tilde forms (`~`, `~user`, `~+`, `~-`, `~N`, named directories) and `EQUALS`' `=cmd` all expand to absolute paths, which no reader takes as a flag.
  `~` exclusion adds no name beyond its literal left side, and qualifiers and `KSH_GLOB` need a parenthesis, which D2 rejects.
  `#` is the gap (F1).
  zsh is not installed on this host, so F1 rests on zsh's documented `EXTENDED_GLOB` semantics, not on a run.

## Prior rounds, read after this review was written

Round 1 and round 2 deferred and disputed nothing, and every round-2 fix is present in the target, so nothing has regressed.
Round 2's review said `x~y` and `x#` "match only names that share the word's literal prefix".
That holds for `~` but not for `#`, since `x#` also matches zero `x`s, which is why F1 is raised now.
Round 2's F4 asked for the dead-fork-at-cap row that F2 finds unreachable.

The orch's note that #518 joins D2's scope is noted, and nothing here flags it as out of scope.
