---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:104-115
    summary: D4 bounds a task-edited publish_verify by claiming the change shows in the PR diff, but landing.yaml lives in the weft repo and never reaches the PR; the only visibility is Finalize's notice after the landing, and an emptied value silently drops the only gate that runs the tmux tier
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:98-115
    summary: D4 moves the tmux tier to Publish alone but does not state that Publish is a no-op for a parent outside require_pr_to_base and Finalize never runs the key, so such a landing runs no tmux gate at all
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:80
    summary: D2's closing bound says a failure only a tmux or llm test shows is caught at Publish, but D1 says no gate runs the llm tier and Publish runs tmux only where publish_verify is set
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:146-148
    summary: D7 says a line is checked when it or the next line is touched and also that untouched lines are never flagged, and D6 and the kept PATTERN text say untouched lines are never rewrapped, so a fix at a touched break forces rewrapping an untouched line
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:149
    summary: D7 justifies a no-base round-gate lint pass with "every card passed it at the card gate", but a round's own fixer commits never went through the card gate, and the no-base case is exactly D2's full-verify fallback
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:129-141
    summary: D6 lists the allowed breaks without stating that a sentence end includes a terminator followed by a closing parenthesis, quote or backtick, which D7 and the acceptance criteria rely on
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:124-125
    summary: D5's bound says Webster's gate catches an unmarked-guard violation, but the Webster-Burler rounds run after Webster's gate, so only Publish or Finalize catches a round's change
  - id: F8
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:169
    summary: D8's tree-sitter guard does not say which build tags its go list runs under, so a package that links tree-sitter only through a tmux or llm test file is undecided, and D10's before and after count has no stated tag set
  - id: F9
    severity: NIT
    class: scope
    location: _lyx/discussion/decision-record.md:119-122
    summary: D5 gives a discriminator and examples for a repo-scanning test but no way to enumerate the existing ones, which the acceptance criterion requires the plan writer to find
---

### [BLOCKING] D4's bound on a task-edited `publish_verify` rests on a false premise

**Location:** `_lyx/discussion/decision-record.md:104-115`, and `_lyx/discussion/support-log.md` Q14.
**Issue:** D4 says a task's edit of `publish_verify` is "additive, never a weakened landing" because "the changed value is in the PR diff and in Finalize's config-change notice".
`_lyx` in a task worktree is a symlink into the weft repository (`test-tiers-weft`), so `_lyx/config/landing.yaml` is committed on the task's weft branch, and the pull request Publish opens is for the warp branch.
`internal/landingshed/doc.go` and `Finalize.configChangeNotice` both state that landing never carries per-worktree config files to the parent.
So the value is not in the PR diff.
The one channel is Finalize's notice, which `pushParent` queues only at Done, after the landing has happened.
"Never a weakened landing" also fails: the plan's `## verify:` no longer runs the `tmux` tier, so Publish's `publish_verify` is the only gate that runs it, and a task that empties the value removes that gate for its own landing.
The full plan verify still runs, which is the Goal's invariant, but the decision claims more than that.
The decision does accept the gap (the value can run any shell command, the same trust class as the plan's `## verify:`), but the bound it names for the gap is wrong, so a plan writer relies on a mitigation that does not exist.
**Fix:** in D4 and in Q14, state the real bound: the edit is visible to the hub only through Finalize's notice after the landing and not in the PR diff; a task can run any command in its own worktree, which `## verify:` already allows, and can empty the value, which drops the `tmux` gate for that landing only; the full plan verify on the landing tree still runs.
State the gap as accepted, in D4 and in Open risks, and remove the PR-diff claim.

### [MEDIUM] D4 does not bound the landings that run no `publish_verify`

**Location:** `_lyx/discussion/decision-record.md:98-115`.
**Issue:** `internal/landingshed/doc.go` says Publish is a no-op, with no merge-in and no verify, when the task's parent branch is absent from `require_pr_to_base`, and D4 says Finalize never runs the key.
A landing into such a parent, or into a trunk a repo left out of the list, therefore runs no `tmux` tier at all, where before this task the plan verify ran it at every site.
D4's rationale, "once per landing is enough", does not hold for those landings, and no bound is stated.
**Fix:** state in D4 that `publish_verify` runs only when Publish does, so a landing into a parent outside `require_pr_to_base` runs none; the bound is that loomyard's `main` is in the list, so anything reaching `main` passes a Publish that runs the key, and a repo that leaves its trunk out of the list has chosen to land without it.

### [MEDIUM] D2's closing bound claims Publish catches a failure only an `llm` test shows

**Location:** `_lyx/discussion/decision-record.md:80`.
**Issue:** "The `tmux` and `llm` tiers are only compiled in a round, so a failure that only one of their tests shows is caught at Publish the same way."
D1's last row says no gate runs the `llm` tier, and D4's `publish_verify` is `go test -tags tmux ./...`, so an `llm`-only failure is caught by no gate.
The `tmux` half holds only where `publish_verify` is set, which is loomyard's config and not the template default.
**Fix:** reword the sentence: a `tmux`-only failure is caught at Publish where `publish_verify` runs the tier; the `llm` tier is run by no gate, by design, so a failure only an `llm` test shows is left to the by-hand run and passes every gate.

### [MEDIUM] D7 and D6 disagree on whether untouched lines are flagged or rewrapped

**Location:** `_lyx/discussion/decision-record.md:146-148` against `:129-141` and the kept `PATTERN-comment-line-breaks` text.
**Issue:** D7's Scope says a line is checked "only when it or the line after it is added or changed", and in the next sentence that "untouched lines are never flagged".
A line whose own text is untouched but whose next line was edited is checked and can be flagged, so the two sentences contradict.
The kept PATTERN bullet, "untouched lines are never rewrapped", then conflicts with the fix: joining a flagged line with its untouched successor rewraps that successor.
D11 says the PATTERN line follows D6 and D7 without saying which wording wins.
**Fix:** in D7, say a break is checked only when the edit touches the line before it or the line after it, and replace "untouched lines are never flagged" with "a break between two untouched lines is never flagged".
In D6, state that the PATTERN bullet becomes: a break between two untouched lines is never a finding, and a fix rewraps only the sentence the edit touches.
D11 then needs no change.

### [LOW] D7's no-base lint pass rests on a false premise

**Location:** `_lyx/discussion/decision-record.md:149`.
**Issue:** "When D2 has no base, the round-gate lint has no diff and passes, since every card passed it at the card gate."
A round's own fixer commits never went through the card gate, and the no-base case is D2's full-verify fallback, after which the new plan-verify pass becomes the next base, so that round's comment edits are never linted.
The pass is a guard bypass with a wrong bound.
**Fix:** state the bound as it is: with no base the lint skips, a comment a fallback round wrote reaches only the reviewers' D6 findings, and the next round's base is that round's pass.

### [LOW] D6 omits the closing-punctuation sentence end

**Location:** `_lyx/discussion/decision-record.md:129-141`.
**Issue:** D6 lists the allowed breaks as semicolon, comma-plus-conjunction and colon.
D7's finding rule and the acceptance criterion also let a terminator followed by a closing `)`, quote or backtick through, and D11 sends the plan writer to D6 for the PATTERN background, so that text would omit a case the lint allows.
**Fix:** add a D6 bullet: a sentence end includes `.`, `!`, `?` or `:` followed by a closing `)`, quote or backtick.

### [LOW] D5 names the wrong gate as the catcher

**Location:** `_lyx/discussion/decision-record.md:124-125`.
**Issue:** "Webster's gate and Publish run the full plan verify ... so it is caught there".
The recipe orders Webster, then Webster-Burler, so Webster's gate has run before the rounds and does not see a round's change; Publish or, for a landing without a Publish, Finalize does.
**Fix:** write "Publish and Finalize run the full plan verify".

### [LOW] D8's guard leaves its tag set open

**Location:** `_lyx/discussion/decision-record.md:169`.
**Issue:** D2 computes the impacted set under `integration,tmux,llm`, and D5 and D9 parse under every tag, but D8's `go list -test -deps` names no tags.
Whether a package that links tree-sitter only through a `tmux` or `llm` test file is outside the allowed list is left open, and D10's before and after count has no stated tag set.
**Fix:** state that the guard and the D10 count run `go list` under `-tags integration,tmux,llm`.

### [NIT] D5 has no stated way to enumerate the existing guards

**Location:** `_lyx/discussion/decision-record.md:119-122`.
**Issue:** the acceptance criterion requires every existing repo-scanning guard to carry the marker, and D5 gives a discriminator and examples.
The `(test)` entries of `PATTERN.md` and the `Enforced by` lines in `pattern/` are the existing register of those tests, and D5 does not point the plan writer at it.
**Fix:** add one clause to D5 naming the enforcing tests of `PATTERN.md`'s `(test)` entries as the starting set.

## Focus departures

None.
Settled against the focus questions: D4 names the trust class and accepts the gap but with a false bound (F1); D5 states its unmarked bound, runs marked tests under `-tags integration` and bars a marker in `tmux` and `llm` files, and its acceptance criterion exists (F7, F9 are the residue); D1's rows and D2's command order agree on tags, and D2's closing bound does not (F3); D6, D7 and D11 disagree as in F4 and F6.
