---
round: 1
focus:
  - "internal/burlerengine/cluster.go splitShellSegments, the new `<`/`>` case and `dropNextWord`: settle whether any redirection form can drop a real word of a mutating git invocation (a redirect as the last token before a `;`, `&&` or newline; `>|`, `&>`, `>&2`, `<<`, `<(...)`/`>(...)` process substitution; a redirect directly followed by a quote), so the guard now passes a command it used to flag."
  - "The same case: settle whether the file-descriptor check (`word` all digits, attached before the operator) mis-handles a digit word that is a real operand, such as `git branch 2 > x`, `git branch 2>x` or a digit word inside quotes, against D1's bound that only the redirection target leaves the words."
  - "cluster_test.go TestFirstMutatingGitCommand: settle whether the new rows cover each D1 and D2 acceptance criterion in the decision record (`git branch 2>&1 | head`, `git 2>/dev/null commit -m x` mutating, quoted `\">\"` operand, the issue #898 chain), and whether any new row duplicates an existing row's coverage."
  - "internal/burlerengine/doc.go guard description and the splitShellSegments doc comment: settle whether each claim matches the code, and whether the docs rule in CLAUDE.md is met for this change."
  - "Scope against the board entry and decision record: settle that the diff leaves the halt kind of a true guard hit (D3) and the `git branch` list-mode classification (D4) unchanged."
---

Seed focus from the diff `db5c48ff..HEAD`: a single commit touching the shell splitter, its table test and `doc.go`.
