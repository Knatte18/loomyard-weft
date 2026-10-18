---
verdict: CIRCLING
rationale: "F1 (design, MEDIUM) sits on pending-flag-protocol-no-bound, open in rounds 1, 2 and 3 and raised again in round 4, so a gating finding recurs on an earlier-open key; no BLOCKING finding remains and F2 (LOW) is a new key, with no relabels."
---

Round 4 carries two findings, neither BLOCKING.

- F1 (design, MEDIUM) maps to `pending-flag-protocol-no-bound`.
  The review itself says earlier rounds each added the rows their lens found and each left another side or field unreached, so the key is unchanged and still open.
  That is the circling condition; the falling finding count is progress elsewhere but does not change the key's status.
- F2 (LOW, design) maps to the new key `card-15-testcommandentry-fixture-seams`.
  It is below MEDIUM and first raised this round, so it is not gating.

The three keys the previous ledger held open besides the pending-flag one (`pattern-md-headroom-overrun`, `command-entry-refusals-no-way-forward`, `command-minimalentries-env-setting-nil`) are not re-raised, and the review's rejected section checks the PATTERN.md arithmetic and the other two areas.
They are marked resolved.

No finding was relabeled and no focus departures section exists.
