---
round: 1
ledger:
  - key: unrecorded-fix-strand-leaks
    rounds: [1]
    status: resolved
  - key: card3-record-batch-330-test-missing
    rounds: [1]
    status: resolved
  - key: pre-spawn-errors-spend-fix-attempt
    rounds: [1]
    status: resolved
  - key: fix-commits-include-parent-merge-history
    rounds: [1]
    status: resolved
  - key: integration-fix-stencil-banner-prompt-file
    rounds: [1]
    status: resolved
  - key: fixed-column-wraps-in-changed-go-comments
    rounds: [1]
    status: resolved
  - key: card2-mid-run-validate-test-not-updated
    rounds: [1]
    status: resolved
---

Round 1 raised seven findings, and the fixer resolved each one in its own commit.
Two were MEDIUM.
The first was a live fix strand leaked when its GUID could not be recorded; it is now removed, and the error names the GUID.
The second was card 3's missing record-batch variant of the #330 test, which now exists.
The five LOW findings were also fixed, each verified against the code:
- pre-spawn preparation now runs before the attempt is recorded;
- fix commits are listed along the first-parent chain;
- the stencil banner no longer claims a prompt file;
- the changed Go comments use semantic line breaks;
- the mid-run validate test has a begun-not-terminal row.
No finding is open.
