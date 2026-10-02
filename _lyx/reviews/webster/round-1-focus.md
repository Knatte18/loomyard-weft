---
round: 1
focus:
  - "Review range is `git diff $(git merge-base main HEAD)..HEAD` (status.json `product.parent` is `main`, merge base a5f0ce156). It spans cards 1-16, the eight round-1 fix commits a prior Webster-Review approved, the clean main merge c0f53d5b4 (its combined diff is empty), and the PR-Rework cards 17-20 (ef3d5e971..8722cd627). Settle whether cards 17-20 keep their stated no-behaviour-change contract against the code cards 1-16 shipped."
  - "Card 18's Move group (commit 1109d5efe): settle whether each of the six files was relocated with `git mv` before its edits, and whether history still follows across the move. Git detects only codebinding_test.go and codecommit.go as renames at the default 50% similarity; code.go, code_integration_test.go, codebinding.go and codebinding_clone_integration_test.go show only at R037-R045. Also settle whether each moved file's leading file comment names its new file name."
  - "Rename completeness across cards 17-19, which do the renames inside Edit and Move groups: settle whether card 20's Verify grep exits clean on HEAD, and whether string-based references the build cannot catch (JSON envelope keys `shortname`/`shortname_recorded`, flag and verb names in help-tree tests, the uncontainedwrite allowlist key, plugin skills, stencils, the sandbox tool) all moved to the new spelling with no consumer left reading the old one."
  - "No-behaviour-change evidence in the renamed tests (agentname_test.go, strand_test.go, fabricengine and fabriccli tests, standalonegeom, hubgeom and reedcli naming tests): settle whether every expected value and assertion is unchanged apart from the spelling, so that no assertion was dropped, loosened or rewritten to fit."
  - "Refusal and help texts: settle whether reedengine/strand.go's no-shortname and invalid-shortname refusals, the fabric clone refusals and bound-weft warning in shortnamebinding.go, the verb refusals in fabriccli/shortname.go, the flag help in fabric.go, and reedcli add.go's and remove.go's help match card 18's and card 19's stated wording, and whether contracts/specs/refusal-spec.md's two rows match the strings the code emits."
  - "Comment-convention compliance on every doc and inline comment changed in commits ef3d5e971..8722cd627, judged against CONSTRAINTS.md's comment line-break rule (semantic breaks, no fixed-column wrap; a line-width finding is never legitimate): settle whether each renamed symbol's doc comment opens with its new name, and whether the doc.go and docs/overview.md edits from card 20 follow the repo's semantic-line-break rule for Markdown and comments."
---

The prior Webster-Review approved cards 1-16 and its own round-1 fixes, so the main new subject is the PR-Rework rename in cards 17-20.
The range is still the whole branch, and a finding in cards 1-16 is still in scope.
That review accepted the `.git`-entry stat in `hubgeom` over card 6's named `PrimeName` mechanism, and that choice is settled.

Card 18 is the only card with a type whose mechanical check this rubric names explicitly (Move: `git mv` plus import fixup), so it gets its own entry.
Cards 17 and 19 rename symbols inside Edit groups rather than a `Rename` group, so the AST-script check does not apply to them as such.
Card 20's Verify grep is the plan's own backstop for leftover spellings, and the string-reference blind spot is what that grep and the build might both miss.
