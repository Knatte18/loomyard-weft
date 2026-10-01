---
round: 2
ledger:
  - key: path-spelled-program-passes-allowlist
    rounds: [1]
    status: resolved
  - key: git-options-run-commands
    rounds: [1]
    status: resolved
  - key: shell-loop-conditional-fails-closed
    rounds: [1]
    status: resolved
  - key: wrapper-shell-option-misread
    rounds: [1]
    status: resolved
  - key: cards-44-46-comment-line-breaks
    rounds: [1]
    status: resolved
  - key: git-grep-pager-abbreviation-fails-open
    rounds: [1, 2]
    status: resolved
  - key: env-assignment-steers-program
    rounds: [2]
    status: resolved
  - key: short-cluster-value-option-misread
    rounds: [2]
    status: resolved
  - key: range-comment-semicolon-line-breaks
    rounds: [2]
    status: resolved
  - key: range-markdown-semicolon-line-breaks
    rounds: [2]
    status: resolved
---

Round 1 reviewed cards 44–46 and the range through them.
Its fixer closed five findings in `a235907f7`..`085c17adb`.
One residual, the `git grep` pager abbreviation, was left open.

Round 2 reviewed `085c17adb` and raised that residual again as F1.
It added two classifier fail-opens: environment assignments that choose which program runs (F2), and short-option clusters whose last letter takes a value (F3).
It also raised two semicolon line-break NITs across the range's Go comments (F4) and markdown (F5).
The fixer closed each finding in its own commit, `118704089`..`a543bf061`.
The judge verified every fix and its test rows at `a543bf061`, where build, vet and both test suites pass.
No entry is open.
