---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "Neither proposed change was made. Moving the cause into preflight.ResolveMode would drop it from every verb that calls lymcwd.Resolve directly. Reporting only the opening text would drop the two paths and break the assertion at internal/driftcli/anchor_integration_test.go:84. The gate now returns an *lymcwd.AnchorGateError, and drift capture-child reports its Refusal field, which keeps the paths and marker without the cause and way forward."
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
---

# Round 1 fixer report

## Changes

- F1 (fixed differently): `internal/lymcwd/anchor.go` adds `AnchorGateError`, which `checkCwdAnchorGate` returns.
  Its text is unchanged apart from F2's quotes, and `errors.Is(err, ErrCwdOutsideAnchor)` still holds.
  `runCaptureChild` in `internal/driftcli/cli.go` reports `Refusal` alone.
  `TestDrift_RunsInTheAnchoredDirectory` now also asserts that the child's error holds no way forward.
- F2: the `cd` target is wrapped in double quotes.
- F3: the root-above-a-subpath-anchor row pins the whole refusal.
- F4: the `lym-shell-directory` entry says a `lym` verb may be refused.
- F5: the pinned row checks the heading level only, with a `Loosened-Check` trailer on its commit.
- F6: the blast radius in `_lym/landing/summary.md` is `agents`.
  Its "What changes" and "Tests" sections now match F1, F3 and F5.
  The file is under `_lym`, so it is not committed by this round.
- F7: the test's doc comment names the whole-text pin.

- Gate repair: a comment line on `AnchorGateError` in `internal/lymcwd/anchor.go` wrapped mid-sentence,
  and is now two one-line sentences.

Commits: `e98d3f114` (F1, F2, F3, F7), `f1ea7a4d1` (F4), `f31436e2c` (F5), the two self-check commits S1 and S2,
then `846436715` (gate repair).

Verification: tier 0 passed for `lymcwd`, `driftcli`, `driftengine`, `heddleengine` and `preflight`.
Tier 1 passed for `driftcli`, `lymcwd`, `preflight` and `heddleengine`, and its log shows `TestDrift_RunsInTheAnchoredDirectory` ran and passed.
Tier 0 for `lymcwd` passed again after S1 and S2, and after the gate repair.

## Self-check

- fix-code-correctness: no findings.
- fix-proposal, LOW (S1): the F7 sentence repeated the table's `wantMessage` comment: fixed as proposed, by removing the table comment.
- fix-consistency, LOW (S2): neither the `Resolve` godoc nor `docs/shared-libs/lymcwd.md` mentioned `AnchorGateError`: fixed as proposed, in both.
- fix-affected-set: no findings.

## Disputed

## Deferred

## Rulings
