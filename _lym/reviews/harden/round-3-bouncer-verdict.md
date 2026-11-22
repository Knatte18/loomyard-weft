---
verdict: CONVERGED
rationale: "Round 3 review raised no finding, the live gate is green, and the reviewer model (sonnet) differs from round 2's (fable); the round-2 fixes F6, F7 and F8 were re-driven on a one-repository sandbox hub and held."
---

## Evidence

- Facts: round 3 carries 0 findings at every severity and class, so the decision rule's gating test finds nothing on any key.
- Live report: green, per the facts and the orch brief (the earlier red was a missing `harden-live` key, restored, rerun green).
- Reviewer models: round 2 `fable`, round 3 `sonnet`.
- Prior fixes: F6 (killed seed push, withdrawal after the read of origin), F7 (prime-key refusals through `RecordsBranchName`),
  F8 (branch merge setting read as a records-store mark) were driven live and resolved.
  The F6 drive was shown to reach the mechanism: the pre-704c7ae build leaves `refs/lym/heads/main-weft` on origin, the current build does not.
- Focus departures: both are ratified.
  The old-build comparison ran by path, since the seed runs in the process of the binary that runs `sandbox new`, so a `PATH` swap would not change it.
  The mutation of the tests was not run because a review may not change source; the tests were read instead.
  That leaves the tests' sensitivity unproven by sabotage, noted under coverage.

## Coverage

Not driven:

- The joined clone, the archive tags, two clones of one origin, lym's own remote delete and a pair verb killed mid-operation need verbs a task session refuses.
  The orch plans them for a second hardener campaign once `harden-sandbox-fabric-verbs` lands.
- The `--reset` way forward in `pushclassified.go` after a killed seeding lym says the board "was never seeded" while origin holds the whole seed.
  The review recorded it as a rejected LOW wording gap it could not drive; it is not a finding of this campaign.
- A kill-and-sleep hook that lets origin apply a push after lym has read it, a stalled git on a remote transport and a killed push onto an origin that already held a records branch could not be built on a local transport.

Method gaps:

- No sabotage run proved the new tests fail without the withdrawal or the read before it; only by-construction reasoning and reading.
- A records store with every mark removed reads as two repositories; the review judged it indistinguishable by settings and outside F8's promise.
- No concurrency claim was made, so no sequential control or second hub was needed.
