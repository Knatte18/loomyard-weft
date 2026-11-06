```yaml
round: 1
fixed: [F1, F2, F3]
```

## Changes

All edits are in `docs/research/codex/review/assist-fan-trial-2026-11-06/report.md`.

- F1: "25,000 in all for the lenses" is now "about 28,000 in all for the lenses" (fresh input plus output, 27,946). Fixed as proposed.
- F2: "17 cases walked" is now "18 cases walked". Fixed as proposed. `data/review.lens-code-correctness.md` is unchanged.
- F3: the unsupported "launched at 11:53:07Z ... on disk at 11:57:3xZ" bullet is replaced by the handler rollout start, 11:53:12Z by its file name, with the 265.8 s `task_complete` turn time as the only timing claim. Fixed as proposed, taking the review's second option and also dropping the equally unsourced 11:57:3xZ figure.

All three fixes went into one commit rather than one per finding: they sit in the same file and the round's edit order forbids running the edits one at a time.

## Self-check

- fix-code-correctness: no findings (prose-only change; figures re-derived).
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
