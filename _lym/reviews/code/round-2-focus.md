---
round: 2
focus:
  - "internal/shedrecipe/entries_burler.go `subjectDiff` and the new committed-diff reader in internal/fabricengine: settle whether the staged patch is the committed `git diff <base> HEAD` range and never the working tree or untracked files, and whether the seam is read once per spawning call and shared by a retried attempt."
  - "internal/burlerengine/diffpieces.go `boundedUnits`, `packUnits` and `pieceFileName`: settle whether every piece stays under both the byte cap and the line cap, whether a split file's later pieces carry the file's header, whether a cut never splits a UTF-8 rune, and whether a deep package path still yields a writable file name."
  - "contracts/stencils/burler/burler-diff-pieces.md against the code-review rubric's range section: settle whether the stencil defers to the rubric where the rubric narrows or refuses the range, and keeps every rule the board entry did not ask to change."
  - "internal/burlerengine/template_test.go, diffpieces_test.go and the assisted-round test: settle whether each kept test carries a `//testtiming:keep` reason naming what it pins, and whether the new rows would fail for the behaviours they claim, namely hunk-first then line-end split order and the assistant's explore step naming the same pieces."
---

Round 2 focus after a CONTINUE verdict.
