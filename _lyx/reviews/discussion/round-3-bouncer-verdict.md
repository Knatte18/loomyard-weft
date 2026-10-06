---
verdict: CIRCLING
rationale: "Round 3 carries a BLOCKING finding F1 (D12 breaches PATTERN-gate-self-check-parity, key d12-gate-parity-claim-without-verb, open since round 2) and a MEDIUM design finding F2 (D1 matching bound, key d1-input-box-matching-rule, open in rounds 1 and 2); both gating keys are still open, so the loop is circling. No finding relabeled and no focus departures."
---

The gating findings of round 3:

- F1, BLOCKING, class `decision`: D12 adds a mechanical gate and no self-check verb, and the Constraints line says the verb half is not delivered.
  It maps to `d12-gate-parity-claim-without-verb`, open in round 2 and open again.
  The pattern entry is gating, so the BLOCKING stands.
- F2, MEDIUM, class `design`: D1's Bound says an extra Enter never lands on a draft of other text, but the contains-match on a short or shared-prefix draft contradicts it.
  It maps to `d1-input-box-matching-rule`, listed by the facts as open in rounds 1 and 2, and open again.
  The round-2 findings on this key were the leading-part length and the read timing after Enter; round 3 finds the match rule itself still unbounded.

F3 (needle length and settle, LOW), F4 (D8 no-turn-end case, LOW) and F5 (D9/D10 reload position and the `doc.go` acceptance line, LOW) are not gating.
F3 is carried under the D1 key, F4 under the D8 key and F5 under the D9 key.
The review carries no `## Focus departures` section.
