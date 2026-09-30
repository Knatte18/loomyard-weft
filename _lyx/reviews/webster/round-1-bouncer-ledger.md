---
round: 1
ledger:
  - key: comment-linebreak-deps-verify-fields
    rounds: [1]
    status: resolved
  - key: comment-linebreak-verifygate-header-check
    rounds: [1]
    status: resolved
  - key: comment-linebreak-publish-step4a
    rounds: [1]
    status: resolved
  - key: comment-linebreak-doc-go-publish-finalize
    rounds: [1]
    status: resolved
  - key: comment-linebreak-loomverify-path-docs
    rounds: [1]
    status: resolved
  - key: comment-linebreak-landingdeps-verifycommand
    rounds: [1]
    status: resolved
  - key: comment-linebreak-new-test-files
    rounds: [1]
    status: resolved
  - key: verifyrun-discard-subtest-misdescribed
    rounds: [1]
    status: resolved
  - key: publish-verify-fixture-unused-ghcalls
    rounds: [1]
    status: resolved
  - key: verifygate-pass-test-writer-not-pinned
    rounds: [1]
    status: resolved
---

Round 1 was the first round.
The Burler review approved with seven LOW findings and three NIT findings.
The LOW findings were mid-sentence fixed-column wraps in new comments, against CONSTRAINTS.md's Comment Line-Break Convention.
The NIT findings were a mislabelled `Discard` subtest, an unused fixture field and a pass test that did not pin the runner's writer.
Ten fix commits, one per finding, resolved all of them, and I confirmed each one against HEAD `a9b2b5fa2`.
The plan's verify block passes.
No finding is open.
