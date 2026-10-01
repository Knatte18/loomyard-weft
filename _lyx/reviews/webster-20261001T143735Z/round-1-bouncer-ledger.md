---
round: 1
ledger:
  - key: shedengine-tests-fail-to-compile-episodestuckcount
    rounds: [1]
    status: resolved
  - key: mutatingcommand-misclassifies-realistic-commands
    rounds: [1]
    status: resolved
  - key: busy-awaiting-way-forward-ignore-runid
    rounds: [1]
    status: resolved
  - key: shed-run-pause-help-carry-goto-admission
    rounds: [1]
    status: resolved
  - key: refusal-spec-cells-paraphrase
    rounds: [1]
    status: resolved
  - key: master-stencil-done-check-antecedent-and-quote
    rounds: [1]
    status: resolved
  - key: gotorequest-producers-doc-stale
    rounds: [1]
    status: resolved
  - key: comment-line-break-violations-cards-32-41
    rounds: [1]
    status: resolved
  - key: goto-entry-fixture-inverts-recipe-order
    rounds: [1]
    status: resolved
  - key: containsname-duplicates-slices-contains
    rounds: [1]
    status: resolved
---

Round 1 reviewed the merge-base diff from `f1bf4c880`, concentrating on cards 32-43, which sit on top of an earlier Webster-Review that approved cards 1-31.
It raised ten findings: one BLOCKING (shedengine's tests did not compile at HEAD), two MEDIUM (quote-blind `mutatingCommand`, way forwards ignoring the told run-id), five LOW and two NIT.
The fixer landed one commit per finding, `9bc9b381a` through `c709c4487`, and the Bouncer verified each at HEAD, with `go vet`, `go test ./...` and the touched packages' integration-tagged suites all passing.
Every entry is resolved, and the verdict is APPROVED.

The verdict file records sub-threshold residue in `mutatingSegment`'s new wrapper handling (a wrapper's separately given option value read as the program).
It is not a ledger entry, since it does not rise to a finding.
