---
round: 1
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
    rounds: [1]
    status: open
---

Round 1 reviewed cards 44–46, commit `8c4b50f2b` and the range through them, after two earlier Webster-Review passes approved everything through card 43.
The reviewer raised five findings against card 44's fail-closed fabric-reference allowlist and the new comments, and the fixer closed each in its own commit (`a235907f7`..`085c17adb`).
The judge verified F1, F3, F4 and F5 as resolved.
F2's `-c`/`--config-env` part is resolved, so `git-options-run-commands` is resolved;
its `git grep` pager part left a residual fail-open, tracked separately as `git-grep-pager-abbreviation-fails-open`:
`grepOpensPager` matches only an `--open` prefix, while git accepts `--op` and `--ope` as unique abbreviations that run the named program.
Build, vet and both test suites pass at `085c17adb`.
