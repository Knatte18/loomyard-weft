---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/shuttleparity/seats.go:126
    summary: The holder glyph internal/landingshed#soloSession.table is written as the same literal in three catalog entries, against PATTERN-code-conventions' rule that a name that can go stale is one named constant.
    fix: In internal/shuttleparity/seats.go declare one constant, such as soloSessionHolder, holding internal/landingshed#soloSession.table, with a one-line doc comment, and use it in the Holders of the DescribeTable, FixerTable and FixReviewSeatTable entries in place of the three string literals.
    origin: assist:assist
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/landingshed/finalize.go#landUnderLock
    summary: The parent-holds-work answer is read once before ownerChecks, whose run-scope reconcile can fast-forward the task branch, so a stale true now skips the land check's blockers and the parent sync that the old per-check reads would still have run.
    fix: In Finalize.landUnderLock keep the one parentHoldsWork read that feeds ownerChecks, then read parentHoldsWork once more after ownerChecks returns nil and pass that second answer to landCheck and syncParent. A failed second read halts with the same OwnerHaltPrefix reason as the first. Correct the summary's sentence that the held read serves checks none of which moves anything, since the owner check's reconcile can move the task branch.
    origin: assist:assist
  - id: F3
    severity: LOW
    class: scope
    location: internal/landingshed/finalize_pr.go#landingSummary
    summary: A missing or malformed change description at Finalize now errors with read the change description instead of parse change description, a behaviour change the board entry's no-behaviour-change constraint does not license.
    fix: Make Deps.landingSummary return the summaryparser.Parse error unwrapped, or with no wording of its own, and wrap it at each caller, so that Finalize.Call wraps it as parse change description and PRReview.pullRequestText as read the change description, exactly as before. Then drop the matching behaviour note from _lym/landing/summary.md.
    origin: assist:assist
  - id: F4
    severity: LOW
    class: scope
    location: internal/landingshed/parentalign.go#ParentAlign.readAttempts
    summary: Parent-Align now moves a scratch attempt record before it reads the row entry, and a failed RowEntry read leaves the record already moved; before, the read came first and failed with nothing moved.
    fix: In ParentAlign.readAttempts read m.deps.RowEntry(parentAlignName) first and return its error, wrapped as landingshed Parent-Align read the row entry as before, ahead of the moveScratchAttempts loop. Give readRowAttempts the resolved entry as a parameter instead of reading deps.RowEntry itself, and have Finalize.readAttempts resolve the entry, 0 with no RowEntry wired, before calling it, so Finalize keeps its order. Drop the matching behaviour note from _lym/landing/summary.md.
    origin: assist:assist
  - id: F5
    severity: LOW
    class: design
    location: internal/landingshed/prreview.go#checkWiring
    summary: The reflection wiring check exempts fields by the names in optionalPullRequestSeams, and no test runs PR-Review with ParentReviewWanted or AskParent unset, so a renamed or dropped name would make the row refuse every run that leaves that seam nil with no test failing.
    fix: In the existing wiring test in internal/landingshed/prreview_test.go, the one whose rows name each unset seam, add a row per entry of optionalPullRequestSeams that leaves only that field unset and asserts that Call returns no wiring error naming it.
    origin: lens:code-tests
  - id: F6
    severity: LOW
    class: consistency
    location: _lym/landing/summary.md:22
    summary: The summary gives production comment lines as 2,568 to 2,570, a figure from commit eb3feb72 that the range's own comment commit 811f9e31 made obsolete; HEAD measures 2,573.
    fix: Re-run go run ./tools/codestats -root internal/landingshed on the final tree after this round's fixes and write the measured production comment count into the sentence on comment lines, and recheck the code-line figures in the table against the same run.
    origin: handler
---

### [BLOCKING] Holder glyph repeated as a literal in the seat catalog

**Location:** internal/shuttleparity/seats.go:126

**Issue:** `pattern/PATTERN-code-conventions.md` says a name or path that can go stale is one named constant, declared once and never a repeated literal.
The diff writes `internal/landingshed#soloSession.table` three times, in the Holders of three catalog entries, at lines 126 to 128.
A rename of `soloSession.table` makes all three stale at once.
The listed-holder scan would catch it, but the rule makes no exception for a tested name, and the instructions make every violation of a listed constraint BLOCKING.
The surrounding catalog's literal-per-entry style does not outrank a written rule.

### [MEDIUM] Parent-holds answer goes stale across the owner check's reconcile

**Location:** internal/landingshed/finalize.go#landUnderLock

**Issue:** `landUnderLock` reads `parentHoldsWork` once and hands the answer to `ownerChecks`, `landCheck` and `syncParent`.
`ownerChecks` with a true answer calls `reconcileVerdict`, and the wired seam (`internal/loomcli/landingdeps.go:135`, `runReconciler`) runs a moving run-scope `ReconcileOrigin`.
That reconcile can fast-forward the run's own branches (`internal/fabricengine/originreconcilemoves.go`, `ffWorktree` and `originMover.move`).
When it moves the task branch past what the parent holds, the old code's own reads in `landCheck` and `syncParent` saw false.
They then checked the beam's blockers and synced the parent.
The new code passes the stale true, which skips both.
A beam can then land while a member is still open, against the spec's land-check rule, and a parent behind its upstream is not fast-forwarded before the parent-side merge.
The summary says these checks are none of which moves the parent, which is true of the parent, but the reconcile moves the task.

### [LOW] Finalize's change-description error text changed

**Location:** internal/landingshed/finalize_pr.go#landingSummary

**Issue:** `landingSummary` wraps the parse error as `read the change description` for both rows.
At Finalize that replaces `parse change description`.
The board entry's constraint is no behaviour change, and its record has no added entry allowing this one.
Sharing the reader does not need the shared wording, since each caller can wrap.
No test asserts either text (`finalize_test.go:454-457` checks only that an error came back), so restoring the old wrap changes no test.

### [LOW] Parent-Align moves the scratch record before a failed row-entry read

**Location:** internal/landingshed/parentalign.go#ParentAlign.readAttempts

**Issue:** The old `ParentAlign.readAttempts` read `RowEntry` first and returned its error before any scratch record moved.
The diff moves the scratch records for both row names first, then reads the entry inside `readRowAttempts`.
A failed entry read now leaves the records directory changed, where it used to leave both places untouched.
The summary names the reorder, but the board entry allows no behaviour change.
The same move makes a nil `RowEntry` read as entry 0 at Parent-Align, where it used to dereference nil.
No Parent-Align test covers that, since its fixture always wires the seam (`parentalign_test.go:149`).
Restoring the read in Parent-Align removes both.

### [LOW] Optional-seam names in the reflection wiring check are untested

**Location:** internal/landingshed/prreview.go#checkWiring

**Issue:** The old switch named each required field in code, so the compiler caught a rename.
The new check reads every `PullRequestDeps` field by reflection and exempts the names in `optionalPullRequestSeams` as strings.
`BeamRecord` and `DropBeamRecord` are nil in the PR-Review fixture, so a slip in their names fails tests.
`ParentReviewWanted` and `AskParent` are always set (`prreview_test.go:780-781`), so a slip in either name passes every test and fails only in production runs that leave the seam nil.

### [LOW] Stale comment-line figure in the change description

**Location:** _lym/landing/summary.md:22

**Issue:** I measured with `tools/codestats` at each commit.
Code lines read 5,418 at the base, 5,312 after `fd8c599d`, 5,213 after `5064cc56`, and 5,131 after `eb3feb72` and at HEAD.
Test code lines read 11,003 at the base and at HEAD.
Those match the table.
Production comment lines read 2,570 after `eb3feb72` but 2,573 at HEAD, after `811f9e31` rewrapped comments.
So the summary's 2,568 to 2,570 is a figure the range itself made obsolete.

## Focus verdicts

- The ending type: every former return path traced to the same outcome, error wrap and cancellation check, including the retry loop, the hand-over and PR-Review's `Call`. The exceptions are F2 and the disclosed text and order changes in F3 and F4.
- The lock-freed, requeue and rebase layer stays. The summary's reason holds: the spec's rule to free the lock before a fixer and to merge anew and re-verify at the front of the queue needs the requeue, and the `LineChange(Baseline, HEAD)` rule needs the Baseline move. Not a defect.
- Removed seams: `parentPuller`, `handedOver`, `parentMerge`, `mergeIntoTask`, `noteFlakyPass` and `creditBlock` had no production caller outside their one site. The merged fixer and fix-review parsers keep each parser's own verdict set, required-section check and error texts.
- `prreview_wait_test.go` changes only its read of the new return. The `seats.go` edit is forced by the listed-holder scan, so the Scope licenses it; F1 is about its form.
- The summary reports code lines per spot, the study's estimate, the spot left unsimplified with its reason and the test lines. The figures check out except F6.

## Rejected

- code-scope 1, `startFailure` outside the Scope: it merges two identical not-started checks in the study's Fixer part. The Goal asks for near-identical helpers merged and the duplicated-logic estimate, and this merge serves that with no behaviour change.
- code-scope 2, `gateCall` and `runMarked` outside the Scope: they merge duplicated command and base reads in the study's Gate and lock plumbing part. Same reason as above.
- code-correctness 1, DescribeTable error precedence: it only decides which of two simultaneous configuration errors is named first. Both are returned errors raised before any session starts, and no spec rule or test defines the order.
- code-tests 2, nil RowEntry at Parent-Align untested: folded into F4, whose fix removes that behaviour.
- code-tests 3, the description error text unasserted: folded into F3.
