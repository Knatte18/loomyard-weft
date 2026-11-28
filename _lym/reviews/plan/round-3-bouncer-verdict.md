---
verdict: CIRCLING
rationale: "F19 (design, BLOCKING) sits on moved-board-tests-duplicate-shared-support, open in round 2 and still open after its fix: card 12 still restates git-using helpers in boardengine_test; F20 and F21 are also BLOCKING design findings, both on new keys"
---

## Why CIRCLING

F19 carries F18, which the round-3 review marks `open`.
It maps to `moved-board-tests-duplicate-shared-support`, which the facts list as open in round 2.
F19 is class `design` and BLOCKING, so it is gating, and the decision rule's CIRCLING condition is met.

The round also made progress.
F15, F16 and F17 are resolved, and F18's notifier fake and fixture types are gone.
What remains on the recurring key is narrower than in round 2: the git-using helpers that card 12 restates.
F19 proposes a way out, which is one exported helper in `internal/gitkit` and an edit to a file of another concern.
The review names that edit as a departure from D9's own-files criterion.

F20 and F21 are new keys and play no part in the CIRCLING call.
All three findings are in card 12.

## Findings and PATTERN entries

F19 cites `PATTERN-test-isolation` and `PATTERN-leaf-packages`, F20 cites `PATTERN-test-isolation`,
and F21 cites `PATTERN-test-stand-ins`.
All stay BLOCKING and `design`.
I relabelled no finding.

The review has no `## Focus departures` section, so there is no departure to ratify.
