---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:77
    summary: "D2's held mark, D2's completeness test and D3's context need to know whether round N-1 routed decisions, a fact the review half and validate-review do not hold"
    fix: "In D1 or D5, record the fact where every reader finds it: the fix producer writes `decisions_routed` (it knows `opts.DecisionsRouted` from NotifyParent or decisions_escalated) into round N-1's D5 snapshot record before it spawns or skips the fixer. State that D2's held mark, D2's test that a fixer-written report passes the fix check, D3's round context and the validate-review builder read it from that record, and that a record without it makes its round incomplete for D2. Add an acceptance row: on a segment whose fix row only notifies the parent (no Bouncer escalation), round N's block marks the decision-class finding as held."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:111
    summary: "A carried finding may change class: nothing in D3 pins it, so a still-open decision-class finding can turn into fixer work, and a design finding can leave the gating class"
    fix: "In D3's `carries` bullet add that a carried finding keeps its earlier finding's class, and add to the round-check entry's refusals a carried finding whose class differs from the one it carries. Add an acceptance row for the refusal."
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:106
    summary: "D3 lets the reviewer mark a held decision-class finding resolved, which drops it from escalation, with no stated bound beyond quoting a listed title"
    fix: "In D3 state what `resolved` can let through and what bounds it. Go checks only that the quoted title is one the block listed, not that the entry addresses that finding, so an unrelated listed entry passes. The bound is the block's content: only entries added since round N-1's snapshot, which the parent itself wrote, can be quoted. Either record that bound as accepted, or tighten the round-check entry so the quoted title must name the held finding's id as a whole word, and make the decision escalation's parent notice tell the parent to start each entry title with that id."
  - id: F4
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:139
    summary: "D3 says validate-review keeps skipping worktree resolution and in the next sentence resolves the worktree from the file's path, without naming the mechanism or the facts the builder needs"
    fix: "Rewrite D3's last paragraph. Narrow the first sentence to the hub, mode and engine wiring the verb still skips. Say the worktree resolves through `lymcwd` only (PATTERN-cwd-resolution), and name the start of that resolution, the seam cwd or the review file's directory; `_lym` is a link out of the worktree, so a path through it does not lie inside the warp. Name the facts the builder needs beside the run directory and round: the decision record path, the anchor path for the parent-review files, and F1's routed flag. Add that the verb's help text, which says it needs no git repository, changes, as a Review criterion."
  - id: F5
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:226
    summary: "D6 puts the stalemate check ahead of softCapRule, so it pre-empts that rule's own accept of a round with no design-wrong entry, which the criterion at line 333 does not cover"
    fix: "In D6 state the check's place against softCapRule's accept (ledger with no design-wrong entry, bouncer.go softCapRule). Recommended: in a window-open round whose ledger holds no design-wrong entry the check yields, so the rule accepts as today and the check runs only where the existing code would not accept. Add an acceptance row for an unchanged window round, with the auto-accept setting off and no design-wrong entry, that is accepted by the rule and not escalated. If the stalemate is meant to win, record that as a Decision with its bound and an Auto-mode assumption instead."
  - id: F6
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:284
    summary: "The second open risk accepts a known loop: an answer whose title repeats a folded entry's title is never listed, yet the entry's body and heading are available to key on"
    fix: "In D5 store, beside each title, a digest of the entry's heading and body (discussionparser.AddedDecisionTexts already returns the bodies), and in D2 list an entry whose digest the record lacks. Delete the second open risk and its trailing sentence in D2 about a repeated title, and adjust the D2 acceptance row on replace and add."
---

### [MEDIUM] The routed-decisions fact never reaches D2's renderer or D3's context

**Location:** `_lym/discussion/decision-record.md:77`, with D1 line 38 and D3 line 120.
**Issue:** D1 takes "routes decisions" from `opts.DecisionsRouted`, which `internal/shedadapters/burlerfix.go:75` sets from the fix row's `NotifyParent` and `DecisionsEscalated`.
Only the fix row holds those: `internal/shedrecipe/entries_burler.go:75` and `:130` build the review row with neither, and `decisions_escalated` is a fix-row key.
D2's held mark, D2's test that a fixer-written report passes the fix check (which needs to know whether a decision-class finding owes a disposition) and D3's round context all run in the review producer or in validate-review.
A row that only notifies the parent leaves nothing on disk for them to read, since only the Bouncer's escalation writes a record.
Plan would have to invent where the fact lives.

### [MEDIUM] A carried finding's class is unconstrained

**Location:** `_lym/discussion/decision-record.md:111`
**Issue:** D3 bounds a carried finding's severity and location path but not its class.
A held decision-class finding carried as `design` goes to the fixer on a round that routes decisions and is never escalated again.
A `design` finding carried as another class leaves `GatingClass` in `internal/burlerengine/verdict.go`, which decides when the loop stops.
The `carries` key is a new routing edge, and the Decision states no bound on it.

### [MEDIUM] The `resolved` status has no stated bound

**Location:** `_lym/discussion/decision-record.md:106`
**Issue:** The Decision lets a reviewer take a held decision-class finding out of the parent's escalation by marking it `resolved`.
The only Go check is that the `answer` quotes a title the block listed.
D4 states the equivalent bound for the `origin: parent` tags; D3 states none for this one.
The rubric asks that a Decision introducing such a path says what it can skip and what bounds it.

### [MEDIUM] validate-review's worktree resolution is contradictory and unspecified

**Location:** `_lym/discussion/decision-record.md:139`
**Issue:** Line 139 says the verb "keeps skipping the hub and worktree resolution", and line 141 says it "resolves that directory's worktree from the file's path".
The acceptance row at line 314 mentions "outside any git repository", so a git-reading resolution is meant.
PATTERN-cwd-resolution allows only `lymcwd` for that; `internal/burlercli/validate.go` today reads only the seam cwd and documents that it needs no git repository.
`git check-ignore` reports `_lym/...` as beyond a symbolic link, so resolving from a run directory's path under `_lym` is not obviously the warp worktree.
The builder also needs the decision record path and the anchor path, which "from the run directory and round number" does not supply.

### [MEDIUM] The stalemate check pre-empts the soft-cap rule's own accept

**Location:** `_lym/discussion/decision-record.md:226`
**Issue:** D6 places the check after the soft-cap auto-accept branch of `settleUndecided` and before `softCapRule`, so that "a round the auto-accept takes is accepted".
`softCapRule` (`internal/shedadapters/bouncer.go:1406`) records an automatic accept when the ledger holds no design-wrong entry.
That is the only accept route for a segment with the auto-accept setting off, and the one a replay with `spawned` false takes.
Such a round, unchanged for the set count, now escalates as a stalemate instead of being accepted as today.
The intent in D6 and the criterion at line 333 cover only the first branch.

### [LOW] An accepted open risk leaves a repeatable loop that a digest removes

**Location:** `_lym/discussion/decision-record.md:284`
**Issue:** An answer entry whose title repeats a folded entry's title is never listed, so its held finding is escalated again with the same notice and the parent is not told why.
D5 keys the baseline on titles alone, although the entry bodies are readable (`discussionparser.AddedDecisionTexts` is used in `internal/websterengine/runlevel.go`).
A digest key removes the collision instead of recording it.

## Rejected

- D2 and D5 baseline across `lym loom decision replace`: round N-1's snapshot is a run-directory record written before its reviewer starts, so a replace between rounds leaves it intact, and a new entry under a new title is listed. Not a defect.
- D3's round-check entry and import cycles: the context is a value built in `internal/shedadapters` and passed to `internal/burlerengine`, which already imports neither `shedadapters` nor `burlercli`; the parent-ask ids are readable through `ParentReviewFilesOf` and `ReadParentReport` in `internal/burlerengine/parentreview.go`. No cycle.
- D4's bound on the `origin: parent` exemption: the entry judges `parent_ids` against the ask's report ids and the round's ask, and the record states that tags inside an ask round are trusted; the stated bound and the enforcement match.
- D6's recording of the post-fix id: `settleUnlessRefused` (`bouncer.go:1037`) is reached from the harvest, replay and judge paths, and records ahead of `routeDecisions`. The direct `settle` call at `bouncer.go:506` serves only a round already holding an escalation record or decision file, and a missing post-fix id there counts as changed under D6, so no count tilts toward stalemate.
- D1, D2 and D6 on a fixer-less round: the Go-written reports in `internal/burlerengine/fix.go` (`skippedFixerReport`, `skippedRoutedFixerReport`) are written only when no finding is the fixer's and the gates pass, so empty dispositions are complete, D2 renders the block after them, and D6 counts them as changed.
