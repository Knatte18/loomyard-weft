---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "The premise holds, but the proposed added sentence says a narrower card's break is first caught by a gate of scope all, which is false: a --main-packages batch call (internal/gatecli/scope.go:203) or another scope holding cmd/lym catches it earlier, and the reported failure was caught at a --main-packages gate."
---

# Fixer report: code round 1

## Changes

- **F1, fixed differently.**
  In `_lym/landing/summary.md`, "Remaining gap" now says the plan rule covers only part of the case.
  It adds that a card with a narrower intent can still pass its own gate,
  and that the test then first fails at a later gate whose scope holds `cmd/lym`.
  - Not made: the proposed sentence "is still first caught by a gate of scope `all`".
    `--main-packages` adds every `package main` to a `--card` scope (`internal/gatecli/scope.go:203-212`),
    so the last batch's call runs `cmd/lym` before any gate of scope `all`.
    The summary's own "What was reported" section records the test failing first at such a gate.
  - Not made: the proposed "only when the card's intent spans the repository" wording.
    The next lines of the section already state the rule's condition in the spec's words,
    and an intent can span packages without covering every instance of one kind.

No commit: the only file changed is under `_lym`.
No gate run: no code or test changed.

## Self-check

No fork finding to judge: every lens returned "no findings".

- fix-code-correctness: no findings
- fix-consistency: no findings
- fix-proposal: no findings
- fix-affected-set: no findings

## Disputed

None.

## Deferred with reason

None.

## Rulings

None.
