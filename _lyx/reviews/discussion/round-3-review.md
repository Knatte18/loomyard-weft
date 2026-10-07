---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:116
    summary: "D4 says everything reaching main passes a Publish that runs the key, but landing config is per-worktree and landing never carries it to the parent, so the key reaches main's copy only through a hub re-apply and no stated rollout step exists"
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:79
    summary: "D2 bound says a failure only a tmux test shows is caught at Publish, but a landing into a parent outside require_pr_to_base runs no Publish verify and no tmux tier, which only D4 states"
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:135
    summary: "D2 and D5 bounds promise a catch at Publish or Finalize at the cost of one rework round and say the plan verify runs every test, but Publish and Finalize carry no on_stuck so a failure halts the run for a human, and the plan verify runs neither the tmux nor the llm tier"
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:158
    summary: "D7 checks a break when its line or the next line changed while D6 says a fix rewraps only the touched sentence, so on a legacy wrapped block every fix changes a neighbouring line, makes its break checked and forces rewrapping to the end of the block"
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:162
    summary: "D7 says a fallback round is linted by no gate, but only the no-base fallbacks leave the lint without a diff, and the card-gate-to-Webster-gate fix commits are linted by no gate whatever the base, which D7 does not bound"
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:148
    summary: "D6 replaces a PATTERN bullet that also holds the clause new and changed comments only, which D7 cites, and leaves the first bullet one sentence per line with a further break at a clause boundary reading as required against permitted never required"
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:6
    summary: "Goal says the full plan verify runs only where a tree is about to land and D1 says any gate never runs the llm tier, but Webster's gate and the round gate's fallbacks also run the plan verify and publish_verify is an arbitrary command"
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:146
    summary: "D6 lists a colon sentence end and then adds that a line ending in a colon is also an allowed break, which is the same rule twice"
---

### [BLOCKING] D4's "everything reaching `main` passes a Publish that runs the key" is false

**Location:** `_lyx/discussion/decision-record.md:114-116`, with `:124` and `:238`.
**Issue:**
D4 puts `publish_verify` in `landing.yaml`, which D4 itself calls the task worktree's own tracked config on the task's weft branch, and `landing` is not `HubWide` in `configreg`.
`internal/landingshed/doc.go` and `internal/landingshed/finalize.go` (`configChangeNotice`) say landing never carries a task's per-worktree config files to the parent: the notice tells the hub to re-apply each change meant for the parent on the parent's copy.
So the value the task commits (the card that "sets loomyard's `publish_verify` value") sits on the task's weft branch only.
`main`'s own `landing.yaml` still has no `publish_verify` after the landing, until the hub re-applies it.
Every task forked from the parent before that re-apply carries a file without the key.
`configengine` fills a missing key from the embedded template, which ships it empty (D4), so those tasks' Publish runs nothing extra and no `tmux` tier.
The sentence "Loomyard's `main` is in the list, so everything reaching `main` passes a Publish that runs the key" therefore holds only for tasks whose own copy carries the key.
The Decision never states the rollout step, so a plan writer reading only this record has no card or acceptance criterion that makes the gate effective on `main`.
D4's bound covers a task emptying its own value but not this default-rollout gap, which is the same attack-surface question: what the new key can skip and what bounds it.
**Fix:**
State in D4 that the value reaches the parent's `landing.yaml` only by the hub re-applying it after the landing, per Finalize's notice, and that a task forked before the re-apply, or whose copy lacks the key, runs no `tmux` tier at Publish.
Reword the "everything reaching `main`" sentence to the true condition.
Add the same gap to `## Open risks` and an acceptance criterion or plan note that the rollout (re-apply on the parent's copy, existing in-flight tasks) is part of the task's done state.

### [MEDIUM] D2's bound omits the landings that run no Publish verify

**Location:** `_lyx/discussion/decision-record.md:79-81`.
**Issue:**
D2 says what the import graph cannot see "is caught at Publish before landing" and that "a failure only a `tmux` test shows is caught at Publish where `publish_verify` runs the tier (D4)".
D4 states the exception: Publish is a no-op for a parent outside `require_pr_to_base`, and `internal/landingshed/doc.go` agrees (Done at once, no merge-in, no verify).
For such a landing only Finalize's plan verify runs, which no longer includes the `tmux` tier (D4), so a `tmux`-only failure reaches the parent branch uncaught.
D2's bound sentences state the catch without that exception, and D1's Publish row also names no such case.
**Fix:**
Make D2's two sentences conditional on the parent being in `require_pr_to_base` and point to D4 for the case where it is not, where only Finalize's plan verify catches the failure and the `tmux` tier runs nowhere.

### [MEDIUM] "One rework round" and "runs every test" do not match the recipe or D4

**Location:** `_lyx/discussion/decision-record.md:79` and `:135`.
**Issue:**
D2 says a miss is caught at Publish "costing one rework round".
D5 says Publish and Finalize "run the full plan verify, which runs every test, so it is caught there at the cost of one rework round".
In `contracts/recipes/loom-recipe.yaml`, Publish and Finalize carry no `on_stuck`: their failures "escalate to a human", halt the run, and are fixed forward on the task branch before a resume.
`PR-Rework` is entered only from `PR-Gate`'s rejection, so no verify failure at Publish or Finalize costs a rework round.
D4 also removes the `tmux` tier from the plan verify, and D1 says no gate runs `llm`, so "runs every test" is false for any test in a `tmux` or `llm` file.
An unmarked repo-scanning test in a `tmux` file is caught only by `publish_verify`, and one in an `llm` file by nothing.
**Fix:**
Say the failure halts the run Stuck at Publish or Finalize and is fixed forward, in place of "one rework round".
Replace "runs every test" with "runs the untagged and `integration` tiers", and say a scanning test in a `tmux` file is caught only where `publish_verify` runs.

### [MEDIUM] D7's checked-line scope and D6's rewrap rule cascade on a legacy block

**Location:** `_lyx/discussion/decision-record.md:158` and `:148`.
**Issue:**
D7 checks a break when "it or the line after it is added or changed", and D6 says "a fix rewraps only the sentence the edit touches" and that a break between two untouched lines is never a finding.
Take a legacy fixed-column block where sentences start mid-line.
An edit touches one line, which flags its break against the untouched next line.
The fix rewraps the touched sentence so it ends at a line end, which moves the start of the next sentence onto a new or changed line.
That line is now "changed", so its break against the following line is checked and flagged if it is mid-sentence.
Each fix therefore makes the next break checked, and the only stop is a sentence that already ends at a line end.
So the lint forces rewrapping lines D6 calls untouched, from the touched sentence to the end of the block, in the same diff.
D7's own sentence "a legacy wrapped block is flagged only where an edit reaches it" holds only until the first fix.
**Fix:**
Define the checked set by the break and not by the line: a break is checked only when the newline between those two words is new in the diff, so a line end carried over unchanged by a fix is not new.
State that rule in D7's Scope and in D6's rewrap bullet, and add an acceptance criterion that fixing one flagged break in a wrapped block leaves the block's other breaks unflagged.

### [MEDIUM] D7's "a fallback round" bound is both too wide and incomplete

**Location:** `_lyx/discussion/decision-record.md:161-162`.
**Issue:**
D7 says when D2 has no base the lint passes, and "a comment a fallback round wrote is then linted by no gate".
Among D2's fallbacks, only "no recorded plan-verify pass" and "commit not an ancestor of HEAD" leave the lint without a base.
In the others (`go.mod` or `go.sum` changed, an unmappable file, `go list` failure, a too-long command) the base exists and the lint still runs over base to HEAD, so "a fallback round" is wrong for them.
The bound also misses a gap that holds with a base.
In the recipe order, the card gate lints each card, Webster's gate runs the plan verify only (D1), and a failure there goes back to Merriam, who fixes it through a fixer fork that commits (`internal/websterengine/verifygate.go`).
Those fixer commits come after every card gate and before the pass that becomes the first round's base, so no round's diff contains them and no gate lints them.
The round's own fixer commits are linted, but only when a base exists.
**Fix:**
Narrow the sentence to the two no-base cases, and add the Webster-gate fix commits as a second place no gate lints, with the same bound: only the reviewers' D6 findings reach them.

### [LOW] D6 replaces a bullet whose clause D7 relies on, and leaves a required-reading bullet

**Location:** `_lyx/discussion/decision-record.md:148` and `:211`.
**Issue:**
`pattern/PATTERN-comment-line-breaks.md` has one bullet reading "A line-width finding is never legitimate, and untouched lines are never rewrapped: the rule governs new and changed comments only."
D6 says its new bullet "replaces the bullet 'untouched lines are never rewrapped'", and D7 cites "the rule's 'new and changed comments only'".
A plan writer replacing the bullet whole deletes the clause D7 depends on.
The first bullet, "One sentence per line, with a further break at an independent-clause boundary", reads as required, against D6's "permitted, never required", and D6 does not say it is reworded.
**Fix:**
In D6, name the exact part replaced (the untouched-lines clause) and keep "the rule governs new and changed comments only" and the line-width clause.
Add that the first bullet is reworded to make the clause break permitted.

### [LOW] Universal claims in the Goal and D1 are narrower in truth

**Location:** `_lyx/discussion/decision-record.md:6` and `:47`.
**Issue:**
The Goal says "the full plan verify runs only where a tree is about to land".
D1's second row has Webster's gate run it after the last batch, which is not a landing, and D2's fallbacks run it at a round.
D1's last row says "Any gate" never runs the `llm` tier and D2 says "No gate runs the `llm` tier (D1)", but `publish_verify` runs any shell command, so the claim holds for loomyard's configured value and not for the gate.
**Fix:**
Reword the Goal to say the full plan verify is not run per round except as D2's fallback, and Webster's gate and the landings keep it.
Reword the D1 row and D2 sentence to say no gate loomyard configures runs the `llm` tier.

### [NIT] D6 states the colon break twice

**Location:** `_lyx/discussion/decision-record.md:145-146`.
**Issue:**
The bullet "A sentence end includes `.`, `!`, `?` or `:` followed by a closing `)`, quote or backtick" already puts `:` among sentence ends, and the next bullet says "A line ending in a colon is also an allowed break".
**Fix:**
Drop the second bullet, or limit the first to `.`, `!` and `?` and keep the colon bullet as the colon rule.
