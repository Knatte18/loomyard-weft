# Fixer report — discussion round 1

## Changes

- F1 (fixed as proposed): in `decision-record.md` D5 Bound, the sentence about an older lym process refusing a second overview add is replaced.
  An older process that rewrites reed.json now has a sentence of its own.
  A new sentence names the old-build pawl driver, which calls `orchcli.BeamUp` in-process at its Beam-Up row (`internal/loomcli/pawlwiring.go:106-107`) and, while it lives, can drop the runs strand or replace the status strand with it.
  Another sentence says that a hand `lym loom start` keeps that driver, that a dropped runs strand returns at the next `lym beam start` under the new build (D4), and that a replaced status strand returns at the next loom start (D3).
  That last clause on the status strand goes beyond the proposed text, because the proposal named a way back only for the dropped runs strand.
- F2 (fixed as proposed, then narrowed by the self-check): a D2 acceptance bullet is added.
  With two overview windows and no live agent, a viewer on the lower-index overview window that ends by its strand's removal or its pane's exit moves to the other overview window, never the shell.
- F3 (fixed as proposed): in `support-log.md` Q5, "open risk recorded for task sessions (D3)" now reads "the task-session rename recorded as an auto-mode assumption (D3)".
- F4 (fixed as proposed): D5 Bound now says that while the old-key status strand opened first, D2's focus fallback lands on its window rather than the beam overview, until the healing loom start replaces it.

## Self-check

- fix-consistency, MEDIUM, new D2 bullet conflicts with the replacement rule in D1 and the Constraints: fixed as proposed. The bullet is limited to a window ending by its strand's removal or its pane's exit (`internal/reedengine/replace.go:71,97-98`).
- fix-proposal, MEDIUM, the same D2 bullet conflict: merged with the fix-consistency finding and fixed by the same change.
- fix-consistency, LOW, support-log Turn 4 no longer matches the D5 bound: disputed, `premise false`. Turn 4 records "existing-paths 3 (an older live lym process, dropped key): fixed; D5's bound names both". The bound still names both: the dropped key ("An older lym process that rewrites reed.json drops the order key") and an older live process (the old-build pawl driver). Turn 4 records an earlier turn's fixes, so it would not be rewritten to follow a later round in any case. The "Review rounds" section belongs to the loop owner.
- fix-claims: no findings.
- fix-way-forward: no findings.

## Disputed

- Self-check fix-consistency LOW (support-log Turn 4): `premise false`; the evidence is in the Self-check line above.

## Deferred

None.
