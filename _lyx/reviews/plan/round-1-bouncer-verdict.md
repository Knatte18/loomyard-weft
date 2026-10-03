---
verdict: APPROVED
rationale: "All eight round-1 findings are legitimate and fixed in the current plan (card 9 states and bounds the gone-worktree quiet exception and keeps the run-lock probe on an absent session; card 11 routes Shutdown refusals through teardownRefusal, restates its inherited deletions and fixes the taskWorktreePresent comment; card 6 gains a Bound; card 9 adds the retire-during-wait archive test; card 7 adds the retiring key to reed status), and an independent pass finds no new defect."
---

The round-1 report raised one BLOCKING and seven lower findings against the plan as it stood at review time.
The plan has since been revised, so each finding was checked against the current card files.

- **F1 (gone task worktree counted as quiet).**
  Legitimate: D2's bound was silently widened.
  Resolved: `09-pairteardown-composite.md` defines "gone" as the path not being a registered linked worktree, narrows the ordinary bound to "with the task worktree present", and states the exception as a bounded list (registered-worktree test, exact-name session only, no post-gone write can reach the archive, D6 commits sibling records first).
  Card 13's module doc covers the exception.
- **F2 (absent session drops the run-lock clause).**
  Resolved: card 9 says an absent session satisfies the strand clause only, and the run-lock probe still runs while the task worktree is present.
- **F3 (batten probe refusals bypass `teardownRefusal`).**
  Resolved: card 11's `Shutdown` passes `EndSession`'s error through `teardownRefusal`, which drops `taskAnchor` and names the pair by slug.
  A new `lifecycle_integration_test.go` case covers the Shutdown path, and the `ImpactSummary` covers the signature change.
- **F4 (card 11's "no new deletion" bound).**
  Resolved: the Bound now names the origin task-branch deletion under card 3's gate, the finish path under card 2's bound, and the gone-worktree driver end under card 9's exception.
- **F5 (`EndSessionByName` without a Bound).**
  Resolved: card 6 states what it skips and why, its reach (exact-match session, given socket, server only when empty), and its single caller and condition.
- **F6 (stale `taskWorktreePresent` doc comment).**
  Resolved: card 11 lists `internal/battencli#taskWorktreePresent` in Edit and rewrites both that comment and the one above `Teardown:`.
- **F7 (retire-during-wait archive criterion untested).**
  Resolved: card 9's integration test commits a stop report, marks the stub strand retiring during the wait through `Engine.MarkRetiring`, and asserts the archive tag holds the report.
  `MarkRetiring` is now a `plan:` Create handle on card 7 and in card 9's Uses.
- **F8 (`lyx reed status` lacks the flag).**
  Resolved: card 7 edits `reedCLI.statusCmd` to add a per-strand `retiring` key, with a test through the fake engine seam.

## Focus departures

The report lists none, and adds that the card-13 granularity directive was reached and judged not a defect.
Ratified: the tripwire fails until cards 10 and 11 move both callers off `Topology.Remove` and card 12 deletes `RemovePairBranch`, so it cannot land with card 9, and it enforces the invariant card 13 writes.
Leaving the detached-remover and batten-grace acceptance criteria unpromoted is also ratified: card 7 tests the unknown-guid no-op, card 8 states the new-guid replacement, and card 9's fake test plus card 11's `AbandonedSession` pass-through carry the batten case.

## Independent pass

- Verify coverage: every package a card targets runs under `go test ./...` and the integration line, `battenshed` and `pairteardown` included; `smoke` is only vetted.
- Fidelity: every Decision D1–D10 and every Constraint maps to a card, and no card adds scope beyond the decision record (the `lyx reed status` key is D7's "strand status exposes the flag").
- Attack surface: each new routing edge or removed guard (finish path, origin task-branch deletion, remote sweep, retiring mark, loom start's replacement, lock-free session end, quiet exception) states its bound in its card's Intent.
- No `Custom` cards; `ImpactSummary` lines on `Edit`/`Delete` cards state blast-radius conclusions rather than restating Intent.
