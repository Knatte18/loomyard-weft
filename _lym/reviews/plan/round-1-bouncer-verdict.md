---
verdict: CONTINUE
rationale: "Round 1 carries two BLOCKING design findings: F1 (card 9 misses the lym shed, batten and fabric routes to a strict loom or tapestry load) and F2 (card 8 assumes a board write path that restores and commits, which the cited code does not)"
---

The decision rule gives `CONTINUE` on F1 and F2 alone.
No finding's class or severity is relabelled.

No finding claims a card violates a PATTERN entry's text.
F5 reports statements in `pattern/PATTERN-config-layers.md` that the plan would leave false, so it keeps its own class and severity.

## Focus departures

- **Planned tests across cards 1 to 10: rejected as a settlement of the whole question.**
  The review did not compare `TestEnsure_MovesOldConfigKeysAndMailsTheMoves` and `TestEnsure_CarriesMovedRunSettingsOnTheBoard` with the existing `TestEnsure_*` functions.
  That comparison is still open and is carried into the round 2 focus.
