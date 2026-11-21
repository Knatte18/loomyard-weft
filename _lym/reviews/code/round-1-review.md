---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/burlerengine/prompt.go#fixScopeRules
    summary: The joint-commit rule does not say whether groups chain through a shared finding, so findings A and C that each share a different file with B read as two overlapping groups {A,B} and {B,C}, putting B in two commits.
    fix: "In the FixScopeSource string of fixScopeRules, state that groups chain: a finding whose change touches a file any member of a group changes joins that whole group, so every finding lands in exactly one commit. Update the matching bullet in internal/burlerengine/doc.go (FixScopeSource) and the fixScopeRules doc comment to say the same."
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/burlerengine/prompt.go#fixScopeRules
    summary: The source rules open with an unconditional order to commit each finding before starting the next, and grouping depends on files a fixer learns only while fixing, so the text leaves open what happens when a later finding touches a file an earlier, already committed finding changed.
    fix: "In the FixScopeSource string of fixScopeRules, tell the fixer to form the groups before it starts fixing, from the files the findings' location and fix keys name, and say that a finding whose change reaches a file an earlier commit already changed is committed on its own, never folded into that commit. Mirror the sentence in the doc.go FixScopeSource bullet."
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/burlerengine/prompt_test.go#TestComposePrompt_FixScope
    summary: The overlay subtest's requireNotContains on the phrase commit each fix can no longer fail, since the diff removed that phrase from the source-scope text, and the test's doc comment still claims a commit-per-fix phrasing.
    fix: "Delete the line requireNotContains(t, got, \"commit each fix\") in the overlay subtest, since the \"no git\" checks already pin the branch split, and reword the doc comment's second line to say FixScopeSource's output carries a commit rule and not the overlay-exclusive \"no git\" phrasing."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/burler/burler-step-fix.md:98
    summary: The never-push section still calls the source-scope rules commit-per-fix, while they now also commit a group of findings as one joint commit.
    fix: "Reword line 98 to read: If your fix-scope rules above have you commit on the repo, stay inside the working tree;"
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/burlerengine/verdict.go:66
    summary: The Finding doc comment says Location points at the offending content, while the review format now says location names the site the fix changes, which may lie apart from where the defect shows.
    fix: "Reword the sentence to: Location names the site the fix changes, or where the defect shows when Fix opens with site not found, and Summary states the problem in prose."
  - id: F6
    severity: LOW
    class: consistency
    origin: assist:assist
    location: contracts/stencils/burler/burler-review-format.md:59
    summary: The new rule that location names the fix site gives no way out for a carried finding, whose location must keep its path or move to a file the changes file lists, as burler-step-review.md and roundcheck.go carriedFinding enforce.
    fix: "Append to the new location rule in burler-review-format.md a sentence saying that a carried finding keeps the location path the carry rules allow and names a fix site outside it in its summary and fix."
---

### [MEDIUM] Joint-commit groups do not say they chain

**Location:** `internal/burlerengine/prompt.go#fixScopeRules`

**Issue:** The rule reads "Findings whose changes touch a file in common form one group".
Take A changing `x.go`, B changing `x.go` and `y.go`, and C changing `y.go`.
A and C have no file in common, so the literal reading yields {A,B} and {B,C}, and B cannot sit in two commits.
The focus directive asked exactly this case, and the fixer is left to improvise again, which is the friction the board entry asks to remove.

### [LOW] The commit order leaves an already committed finding open

**Location:** `internal/burlerengine/prompt.go#fixScopeRules`

**Issue:** "You commit each finding's fix on its own, once green, before starting the next finding" is stated before, and without reference to, the group rule.
Which files a fix changes is often known only while fixing.
The text does not say whether the groups are formed up front or what a fixer does when a later finding touches a file an earlier commit changed.

### [LOW] The overlay subtest guards a phrase no scope renders

**Location:** `internal/burlerengine/prompt_test.go#TestComposePrompt_FixScope`

**Issue:** `requireNotContains(t, got, "commit each fix")` at `prompt_test.go:355` was a guard against the source rules leaking into the overlay prompt.
The diff reworded the source sentence to "commit each finding's fix", so the assertion is vacuous.
The doc comment at `prompt_test.go:330` still names a "commit-per-fix phrasing" the source subtest never asserts.

### [LOW] The fix stencil still calls the source rules commit-per-fix

**Location:** `contracts/stencils/burler/burler-step-fix.md:98`

**Issue:** "If your fix-scope rules above name commit-per-fix on the repo" no longer describes the rules, which now also commit a group of findings together.

### [LOW] The Finding doc describes the old location meaning

**Location:** `internal/burlerengine/verdict.go:66`

**Issue:** "Location points at the offending content" contradicts the review format's new rule that `location` names the site the fix changes.

### [LOW] Fix-site location conflicts with the carried-finding path rule

**Location:** `contracts/stencils/burler/burler-review-format.md:59`

**Issue:** A carried finding must keep its location path or move to a file the changes file lists (`burler-step-review.md:51`, enforced by `roundcheck.go`'s `carriedFinding`).
When a later round finds the fix site in an unlisted file, the new rule and the carry rule cannot both hold, and the round check refuses the move.

## Rejected

- Assistant F2, a missing `Root-Cause:` trailer: the board entry is kind `task`, labelled `enhancement`, not `bug`, and asks for a new commit rule and a review-format change, not for a code defect to be removed, so the rubric's bullet does not apply.
- `burler-step-fix.md:26`'s rule that the files to change are those `location` names, read against the new marker: the next sentence qualifies it directly, and the three dispositions are untouched, so no conflict remains.
- No coverage for the new prompt text: the joint-commit rule is static prose returned by a fixed branch, exempt under the test-economy bullets, and the branch split stays pinned by the `"no git"` checks.
- No commit subject parser needs a change: no Go code reads a single finding id from a commit subject.
- The `doc.go` FixScopeSource bullet matches the prompt text, including the group subject format, and documents the package's own behaviour, in line with `PATTERN-code-conventions`.
- Scope: both "What to do" points are built, and the entry's "Separately" notes lie outside them, as the change description says; no commit claims an improvement or agreement.
