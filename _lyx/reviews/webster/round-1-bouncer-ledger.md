---
round: 1
ledger:
  - key: foreign-probe-doc-comment-not-moved
    rounds: [1]
    status: resolved
  - key: comment-line-break-convention
    rounds: [1]
    status: resolved
  - key: refusal-rows-assert-no-sender-no-driver-log
    rounds: [1]
    status: resolved
  - key: dirof-duplicates-filepath-dir
    rounds: [1]
    status: resolved
  - key: midmerge-foreign-probe-error-unwrapped
    rounds: [1]
    status: resolved
  - key: test-header-becomes-package-doc
    rounds: [1]
    status: resolved
---

Round 1 is the first round.
The reviewer raised six findings: two MEDIUM (the doc comment that did not move with the extracted `readForeignProbes`, and fixed-column comment wraps), two LOW (the sender and driver-log assertions missing from refusal rows, and the hand-rolled `dirOf`) and two NIT (the unwrapped foreign-probe error, and the test header becoming a package doc comment).
The fixer landed one commit per finding.
The judge checked every fix in the diff at HEAD and re-ran the build, vet, unit and integration checks, and all six are resolved.
No finding remains open.
