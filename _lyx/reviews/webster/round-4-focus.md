---
round: 4
focus:
  - "Comment line breaks: list every `//` line the branch adds over `git diff $(git merge-base main HEAD)..HEAD -U0` in Go files, group them into comment blocks, and check each sentence in each block against `PATTERN-comment-line-breaks` (one sentence per line, a further break only at an independent-clause boundary, never mid-phrase, never a fragment line). Settle for every block, not a sample, whether any sentence the diff added or rewrote is still split mid-phrase or leaves a fragment line, including blocks in new files, test files and `doc.go`. Sentences in an untouched wrapped comment that only swap or lose words are exempt."
  - "Rewritten tails: for each comment the diff edits rather than adds, compare the edited sentence with its base version and settle whether the clause was rewritten (then the whole sentence must satisfy the line-break rule) or only had words swapped or deleted (then it is exempt)."
---

The focus is one method applied exhaustively: derive the added comment lines mechanically rather than by eye, then check each sentence.
