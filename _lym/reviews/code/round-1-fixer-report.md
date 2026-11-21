---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
---

## Changes

- F1, F2: the `FixScopeSource` string of `fixScopeRules` now says to form groups before fixing, that groups chain, and that a finding reaching an already committed file is committed on its own.
  The commit-order sentence opens with "Except for a group below".
  The `fixScopeRules` doc comment and the `doc.go` `FixScopeSource` bullet carry the same sentences.
  Commits `ced92802` and a follow-up commit.
- F3: removed the vacuous `commit each fix` assertion and reworded the `TestComposePrompt_FixScope` doc comment.
- F4: `burler-step-fix.md` never-push section now reads "have you commit on the repo".
- F5: the `Finding` doc comment says `Location` names the site the fix changes, one sentence per line.
  A follow-up commit split the comment's first draft, which wrapped mid-sentence, at sentence boundaries.
- F6: `burler-review-format.md` location rule gains the carried-finding sentence.

Gate: `lym gate test ./internal/burlerengine` passed, 634 passed and 0 failed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal, LOW: the unconditional commit-order sentence contradicts the group rule. Fixed as proposed.
- fix-proposal, LOW: transitive chaining can collapse a review into one commit. Disputed, `every fix worse`.
- fix-affected-set, LOW: `contracts/recipes/tapestry-recipe.yaml:452` still says "commit-per-fix". Disputed, `premise false`.

## Disputed

- fix-proposal chaining cost, `every fix worse`: the only alternative to chaining puts a finding in two commits, which one file's changes cannot be split into (`internal/burlerengine/doc.go` FixScopeSource bullet).
- fix-affected-set `tapestry-recipe.yaml:452`, `premise false`: the comment is a loose label for the Fabric Git Invariant's permitted agent commit, and a per-finding commit remains the default it names.
  Neither the review nor the rubric names it as stale.

## Deferred

None.

## Rulings

None.
