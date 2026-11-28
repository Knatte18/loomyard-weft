---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: contracts/stencils/loom/loom-block-plan-cards.md:56
    summary: "The first-caller rule rests on a false premise and contradicts the PATTERN-test-only-code background: the reachability guard carries //lym:only upkeep, so no card gate runs it, and the background says a card may add a function that only a later card calls."
    fix: "Remove the first-caller rule from all three places it lands, restoring each to its base text. In contracts/stencils/loom/loom-block-plan-cards.md, delete the paragraph at lines 56-58 (from 'A new production function goes on the card that adds its first production caller' to 'A test-only caller does not count.') with its trailing blank line. In contracts/stencils/loom/loom-rubric-plan-review.md, delete the bullet at lines 58-62 headed 'A new production function and its first caller.' In contracts/specs/loom-plan-spec.md, delete lines 366-368 (from 'A new production function is the exception' to 'A test-only caller does not count.'). Keep the card-claims and cards-against-each-other text. In _lym/landing/summary.md, drop the first-caller bullet under ## What changes, the spec bullet, and the two ## Design decisions bullets about the first-caller split. Add one line saying #290's ask is already met: commit 2326ef4da moved the guard to upkeep and removed the create-caller-later check, so a card gate no longer fails on a function whose first caller comes later. Facts read: cmd/lym/testonlycode_linux_test.go:90-92 (//lym:guard, //lym:only upkeep), pattern/PATTERN-test-only-code.md:30-32, git show 2326ef4da (section 'Plan check removed'), and git merge-base --is-ancestor 2326ef4da 5c985d926, which succeeds."
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:code-correctness
    location: contracts/stencils/burler/burler-step-fix.md:29
    summary: "The unverified-fact line lets the fixer either make its own change or defer when the code disproves the fact. That leaves out Dispute (premise false), which the ordered dispositions below put first and which is the right outcome when the disproved fact is the finding's own premise, as in the #288 SharedHub case."
    fix: "Replace lines 29-30 with: 'A `fix` key that names a fact as unverified rests on that fact: read it in the code before you apply the change, and give the finding its disposition below with what you read.' This sends a disproved premise to Dispute and a disproved fix-only fact to Fix with your own change, or to defer when no change inside the write surface exists, without a second rule beside the ordered list. Fact read: burler-step-fix.md:31-50."
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/burler/burler-review-format.md:37
    summary: "'Never write a fix that rests on a fact the code disproves' has no route for a kept parent finding. Parent consolidation makes that finding's fix carry the parent item's change, and the reviewer may never reject it."
    fix: "Below line 40 of burler-review-format.md, add a bullet: 'A finding whose `fix` carries another reviewer's change it may not reject, such as a kept parent finding, keeps that change and appends the fact the code disproves, or the fact left unverified, naming the code read.' Leave burler-parent-consolidation.md unchanged. Facts read: burler-parent-consolidation.md:20 and :23."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: contracts/stencils/burler/burler-review-format.md:39
    summary: "The new route that raises a finding as class decision when no fix holds cannot apply to a carried finding, which keeps its class (line 83), and the text does not say what such a carried finding does."
    fix: "At the end of line 39, after 'or build what is missing.', add: 'A carried finding keeps its class, and its `fix` names both choices instead.' Fact read: burler-review-format.md:81-83."
---

### [BLOCKING] The first-caller rule rests on a disproved premise and contradicts PATTERN-test-only-code

**Location:** `contracts/stencils/loom/loom-block-plan-cards.md:56`. The same rule is at `contracts/specs/loom-plan-spec.md:366-368` and `contracts/stencils/loom/loom-rubric-plan-review.md:58-62`.

**Issue:** The diff tells plan writers and Plan-Review that a new production function must share a card with its first production caller, "since a check that every production function is reachable from a `main` package can fail on any card between the two."
No card gate runs that check.
`cmd/lym/testonlycode_linux_test.go:91` marks `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` with `//lym:only upkeep`.
`pattern/PATTERN-test-only-code.md:30-32` says no task's gate runs it and "A card may therefore add a function that only a later card calls."
Commit `2326ef4da`, an ancestor of the review base, deliberately removed `create-caller-later` "with its spec text and the stencil rules that went with it", because "No card gate runs that guard now."
The diff brings that rule back against a repository rule document, and the constraint wins over the board entry's request.
Issue #290 predates that move, and the move already meets its ask.
The rule also has a side effect: moving a function past an earlier card whose test calls it makes that test use a later card's target, which `uses-later-target` refuses.
Commit `e489adf46`'s message and `_lym/landing/summary.md` repeat the false premise.
The commit message cannot be amended within this round, so the fix corrects the summary.

### [MEDIUM] The fixer's unverified-fact line leaves out Dispute

**Location:** `contracts/stencils/burler/burler-step-fix.md:29`

**Issue:** When the code disproves a fact that a `fix` names as unverified, the line allows only "fix with your own change or defer".
When that fact is the finding's premise, the ordered dispositions right after it require Dispute with `premise false` first.
That is what the fixer rightly did in #288's `SharedHub` case.
The two rules in the same block disagree.

### [MEDIUM] A kept parent finding cannot obey the new disproved-fact rule

**Location:** `contracts/stencils/burler/burler-review-format.md:37`

**Issue:** `burler-parent-consolidation.md:20` makes a kept parent finding's `fix` carry the parent item's `change`, and line 23 forbids rejecting it.
The new rule says never to write a `fix` that rests on a disproved fact.
A reviewer holding a parent change that the code disproves has two instructions that cannot both be followed, and no way to flag the fact to the fixer.

### [LOW] A carried finding cannot take the new decision route

**Location:** `contracts/stencils/burler/burler-review-format.md:39`

**Issue:** The rule raises the finding "as class `decision`" when no fix holds without the disproved fact.
A carried finding keeps its class (line 83), so a later round that finds a carried `design` finding's fix disproved has no stated path.

## Rejected

- code-affected-set F2, the false premise in the commit message and summary: merged into F1, since it is the same defect and F1's fix corrects the summary.
- code-affected-set F4, the rubric's granularity bullet not marking the first-caller exception: moot once F1 removes the rule.
- code-correctness F4, the first-caller move creating a `uses-later-target` refusal: merged into F1 as a further consequence of the same rule.
- Handler, `## Merge danger` blast radius "stencils" leaves out the spec: `loom-plan-spec.md` is deployed prose read by the same agents, the two-way call holds, and the diff changes no record format, config meaning or data.
- Handler, a #291 case where one card contradicts itself: the entry asks only to check cards against each other and against the code, and both are carried.
