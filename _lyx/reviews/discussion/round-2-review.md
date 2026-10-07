---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:166-169,208
    summary: D13 and the Constraints demand one weft-path and spelling rule shared by planparser and the audit, but D13 describes a narrower rule than the audit's matcher and names no home that planparser may import
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:181-185,190
    summary: D14 calls its reader list closed yet gives it only as such-as examples, and members like sort and uniq can write a file, so the claim that the commands could not write is not bounded
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:179-185
    summary: D14 bound says the evidence shows the task tree is the batch's own commits, but the start commit being an ancestor of HEAD does not show that
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:141-148
    summary: D11 says one run can contribute batches to several profiles and also says a multi-profile run is reported in its own row and never split, leaving open which figures reach a profile's summary
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:145-146
    summary: D11 marks a --fresh run but does not say whether its figures enter the per-profile summary, nor how weight over all attempts is paired with cards and findings of the final attempt only
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:143-144,147
    summary: D11 counts cards from the recorded partition but also counts a run with no recorded partition as identity, so the card count of such a run has no stated source
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:166,231
    summary: D13 refuses a weft path in an inline code span but exempts the fabric spelling there with no stated reason for the asymmetry
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:10,87
    summary: Goal and D5 state guarantees stronger than the bounds in D13, D14 and D4 (no command reaches the fabric repo, never a batch over budget)
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:71,140
    summary: D4 says the operator reads an over-budget lone card from the calibration report, but D9 to D11 report no recorded estimate or over-budget marker
---

### [MEDIUM] D13's shared matching rule is narrower than the audit's matcher and has no importable home

**Location:** `_lyx/discussion/decision-record.md:166-169` (D13), `:208` (Constraints), `:28` (Out of scope).
**Issue:** D13 defines the weft match as "a path segment ending in the weft suffix after a separator" plus the `lyx fabric|weft|warp` spelling in command position.
The Constraints require that rule to be "one rule shared by the plan check and the implementer audit; neither keeps its own copy, and `planparser` takes no import of an engine package for it".
The audit's matcher, `fabricengine.RefScanner` (`internal/fabricengine/refscanner.go`), matches more: the exact `WeftWorktree(l)` path, which needs a `*lyxcwd.Location`, and a bare word ending in the weft suffix outside quotes.
So the record is inconsistent three ways.
If the shared rule is the audit's, the plan check also needs a location (told-geometry for `planparser`) and refuses bare words, which D13's bound does not mention.
If it is D13's narrower rule, the audit's matcher changes, which the Out of scope line "Any change to what the implementer audit flags" excludes.
And no package is named that both can import: `planparser` imports only `lyxdirs` and `glyph`, `internal/weftname` is imported only from the owner set (`PATTERN-fabric-vocabulary`), and webster reaches the matcher through an injected seam.
This is a design conflict between the record's own constraints, not a call-site enumeration.
**Fix:** State in D13 which rule is shared (the audit's full matcher or a stated subset), whether the audit's flags change, and the shape of the seam (an injected matcher on the plan-format validation, or a named leaf home), without naming signatures.
Reconcile the Out of scope line with the answer.

### [MEDIUM] D14's reader list is not closed and not write-free

**Location:** `_lyx/discussion/decision-record.md:181-185`, rationale at `:190`.
**Issue:** D14 says a command is read-only when built "only from a closed list of reader commands (such as `cat`, `head`, ...)".
"Such as" makes the list open, so the plan writer must choose its members, and AC wording ("an unlisted command") depends on that choice.
The listed examples are not all write-free: `sort -o FILE` and `uniq INPUT OUTPUT` write a file, and `sort --compress-program` runs a program.
The exclusions given (redirection, `tee`, substitution, `xargs`, `find -exec`, in-place flags, interpreters) do not cover them.
The rationale rests on "a command that cannot write cannot have rewritten run state", and the Bound repeats "the commands could not write".
The Decision widens a correctness route, so its bound must hold for what it admits.
**Fix:** Make the list closed (enumerate the members) and state per-member argument limits or drop members that can write, so the bound "could not write" is true of every admitted command.

### [LOW] D14's evidence claim overstates what ancestry shows

**Location:** `_lyx/discussion/decision-record.md:179-185`.
**Issue:** The Bound says "The evidence shows the task tree is the batch's own commits".
The kept evidence is that the recorded start commit is an ancestor of HEAD and the worktree is clean.
A recorded start is an ancestor of HEAD unless history was rewritten, so it does not show that HEAD..start holds only the batch's own commits.
The previous rule (HEAD equals the start) did show an unchanged tree; the widened rule shows less, and the record says it shows the same.
**Fix:** State what the ancestor check actually bounds (for example only that history was not rewritten), or add the evidence that makes the sentence true; drop the claim otherwise.

### [MEDIUM] D11 contradicts itself on a run whose batches name several profiles

**Location:** `_lyx/discussion/decision-record.md:141-148`, acceptance criterion at `:249`.
**Issue:** D11 says a rebaseline "gives the tail the then-active one, so one run can contribute batches to several profiles", and that failed batches and recoveries are "attributed per batch".
It then says Webster-Review findings are per run and "a run whose batches name more than one profile is reported in its own row, never split between profiles".
The record does not say whether such a run's batches, cards and weight enter each profile's summary line or only its own row.
The two readings give different per-profile weight per card and different failed-batch counts, and the focus question is exactly that this run is not split silently.
**Fix:** State one rule for a multi-profile run's contribution to every per-profile figure (all in its own row and none in the profile lines, or per-batch figures split with the per-run figures left in its own row), and make the acceptance criterion test it.

### [MEDIUM] D11 does not handle an earlier attempt's weight, cards and findings together

**Location:** `_lyx/discussion/decision-record.md:145-146`.
**Issue:** For a run restarted with `--fresh`, D11 says the last state "shows only its final attempt, so its row is marked with that reason".
Cards, failed batches and recoveries then come from the final attempt only, while weight comes from fork transcripts, and nothing says whether those cover earlier attempts' forks.
Weight per card of a restarted run would divide all attempts' weight by the final attempt's cards.
The record also does not say whether a marked row enters the per-profile summary or is shown beside it.
The review files have the same gap: only `prior-generation/` is excluded, and the attempt a review file belongs to is not tied to the last state's cards.
**Fix:** Decide whether a marked run's figures enter the profile summary; if they do, state which attempt each figure covers and pair weight, cards and findings from the same attempt, otherwise exclude the run from the profile lines and keep it as a marked row.

### [LOW] D11's card count has no source for a run without a recorded partition

**Location:** `_lyx/discussion/decision-record.md:143-144,147`.
**Issue:** "cards are counted from the recorded partition", yet "A run with no recorded partition counts as `identity`".
For such a run the partition does not exist, so the card count that divides weight and findings has no stated source (the plan commit that D8 already names is the likely one).
**Fix:** Name the card count's source for a run with no recorded partition.

### [LOW] D13 refuses a weft path in an inline code span but not the spelling, without a reason

**Location:** `_lyx/discussion/decision-record.md:166`, Open risks at `:231`.
**Issue:** The spelling is exempt in inline spans "so a card that documents such a command in an inline span is not refused".
The weft path in an inline span is refused, and the Open risks admit that this refuses a card that documents a path.
The record gives no reason the two matches differ, and the support log rejects scanning prose because documenting cards would be refused, which is the same outcome.
**Fix:** Give the reason for the asymmetry in D13, or scan both matches over the same text.

### [LOW] Goal and D5 state stronger guarantees than their bounds allow

**Location:** `_lyx/discussion/decision-record.md:10`, `:87`.
**Issue:** The Goal says a card "can no longer order a command that reaches the fabric repo without the plan gate refusing it"; D13's bound lists what it misses (`bash -c`, variables, absolute paths, `..`).
The Goal also says `accept-audit --batch` no longer discards a committed batch whose findings are read-only commands; D14 only admits a closed list.
D5 says a too-high position gives "more splits, never a batch over `budget`", while D4 lets a one-card batch exceed it.
**Fix:** Word the Goal to the refused and accepted classes the Decisions define, and limit D5's sentence to multi-card batches.

### [LOW] D4 points the operator to a report that does not show an over-budget lone card

**Location:** `_lyx/discussion/decision-record.md:71`, `:117-140`.
**Issue:** D4 says the operator "reads it from the calibration report".
D10 recomputes estimates with the named profile at the session position, and D11's per-batch table lists measured peak and weight; neither shows the recorded `Estimate` or flags a lone card above `budget`.
**Fix:** Either add the recorded estimate or an over-budget marker to the per-batch table, or change D4 to say the operator reads it from the batch's recorded `Estimate` in state.json.

## Checks that found no defect

- Card commands reaching the fabric repo: D13, D14 and D15 each carry a Decision and an acceptance criterion; D13 and D14 state bounds (F1 to F3 correct their content); D15 adds no guard.
  The Out of scope list no longer names the three imported options as wanted; its audit-flags line is the one F1 contradicts.
- Position-dependent start: D4 and D5 each state what is let through and what bounds it; F8 and F9 correct wording only.
- The rubric's three "do not flag" items and the writer stencil's bounded-exploration rule were respected: no finding asks for a rejected alternative, a call-site list or a signature.
