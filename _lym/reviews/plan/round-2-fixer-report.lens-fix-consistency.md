```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens, checking whether the round-2 changes agree with sibling cards, decisions and cross-references.

**Findings**

1. LOW, `_lym/plan/01-hub-registry.md:51`, none refusal now `way forward: run lym creel inbox in that hub once`.
   - The twin in the decision record is unchanged, so the card now departs from it with no recorded departure.
   - `_lym/discussion/decision-record.md:37` still reads "run any `lym creel` verb, such as `lym creel inbox`, in that hub once".
   - The new text is a true subset and meets the acceptance line at `decision-record.md:205` ("naming `lym creel inbox` in that hub"), so no behaviour is wrong.
   - `00-overview.md:43` already says "D1 names `lym creel inbox`", but nothing in the overview says the card dropped the "any verb" wording or why (`lym creel hubs` records nothing).
   - The record is outside the fixer's write surface. An overview note, or a line in the fixer-report, would close the gap.

2. LOW, `_lym/plan/08-outbound-worker.md`, added sentence "An answer with neither `accepted` nor `refused` also gives `bad-answer`, with none of its text in the reason", and the row saying the same.
   - D2 at `decision-record.md:62` gives two forms: a code "for a failed connection or a timeout", and "for an answer of the receiving daemon it is that answer's text", quoted.
   - A neither-word answer is an answer of the receiving daemon, yet the fix gives it a code with no text.
   - The code is the safer reading, and the existing `bad-answer` mapping for a hub-path mismatch already departs the same way.
   - The fix widens that departure, and no plan text records it. A grep of `00-overview.md` finds no `bad-answer`.

**Checked and agreeing**
- `readLine` (`internal/creelengine/protocol.go:135`) needs a `\n`, which matches card 7's "written verbatim with a line end".
- `listenHubFake`'s new signature appears only in card 7's Create line. Cards 0–6, 9 and 10 do not mention it, and card 8 lists it without a signature.
- Card 7's existing `hubFake` rows and card 8's rows still read correctly under the two-value `answer`.
- Cards 5 and 2 moving `docs/overview.md` to Edit agrees with card 10, which already lists it under Edit. Card 1 keeps Extends correctly, since it only adds a line.
- Card 4's `registryErr` wording and card 5's raw propagation describe different callers and do not conflict.
- The rewritten over-200-byte row agrees with the id-clash row and with line 51's return at once.

**Tool calls**
- Grepped cards 00, 03, 06, 09 and 10, and listed `_lym`: those cards and the decision record were not in context.
- Grepped `decision-record.md`, the other cards' doc refs, and `readLine`: none of these were in context.
