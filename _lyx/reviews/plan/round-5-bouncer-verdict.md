---
verdict: CIRCLING
rationale: "Round 5 F1 (BLOCKING, design) is the same enumeration gap as the key clone-output-tests-unplanned-after-card-15, open in round 4 and still open: card 15 leaves clone tests that read per-worktree files unlisted or without a stated expectation; F2 and F3 are new consistency keys, no relabels."
---

F1 maps to `clone-output-tests-unplanned-after-card-15`.
Round 4 raised it for `TestNewHub`, two `TestRunCLI_CleanHubScenario` steps and the `TestOrchIntegration_Refusals` subtest.
Round 5 finds three further integration-tagged clone tests with the same defect, so the enumeration method is still incomplete.
It is gating (design, BLOCKING) and sits on a key the facts list as open in an earlier round, so the verdict is `CIRCLING`.

F2 (`reseed-case-collides-with-newstalehub-migration`) and F3 (`hubreconcile-test-edits-not-member-glyphs`) are new consistency keys, non-gating.

The round 4 keys `card16-newstalehub-order-against-seedretiredkey`, `merge-record-uncleared-before-next-walk-assertion` and `second-ensure-meets-current-stamp-no-walk` draw no finding in round 5 and are resolved.

No finding was relabeled and there are no focus departures.
