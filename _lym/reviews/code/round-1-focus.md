---
round: 1
focus:
  - "internal/burlerengine/prompt.go#fixScopeRules: does the joint-commit rule give a fixer an unambiguous grouping, including a finding that shares a file with two otherwise unrelated findings, and does it keep the per-finding rule for the rest?"
  - "internal/burlerengine tests reaching fixScopeRules: does any test assert the new joint-commit prose or the changed commit sentence, or does the diff add behavior with no coverage? Check the test-economy bullets before filing."
  - "contracts/stencils/burler/burler-step-fix.md and burler-review-format.md: does the `site not found —` marker agree across the review-format stencil, the fix stencil and the fix-scope rules, and does it conflict with the fix step's three dispositions or its rule that the files to change are the ones `location` names?"
  - "internal/burlerengine/doc.go FixScopeSource bullet: does it match the prompt text in `fixScopeRules`, including the commit message format for a group?"
  - "Commit messages of the range: the answer key is a bug-style board entry with no Scope or Decisions; check whether a `Root-Cause:` trailer is required and whether any claim of improvement lacks figures."
  - "Scope against the board entry: it asks to decide the commit rule for shared files and to let `location` and `fix` carry the seam; check the diff makes both changes and no change beyond them, and whether any other stencil or consumer of the `location` key or the commit-message format (such as a parser or a gate that expects one finding id per commit subject) needs a matching change."
---

Seed focus from the diff over `86145ecf..HEAD`: two stencil edits, one prompt function and its doc.
