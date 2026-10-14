# Fixer report — discussion round 2

Every finding's premise held against the code: `configsync` migrates per document, `ArbiterEnvelope.Cause` carries the verb's hand-on cause, `shedengine.Goto` appends an entry with no output, the verdict words are `resume` and `hand-on`, and `configreg.PrepareRunSettings` runs `configengine.ValidateLocal`, which refuses guard keys.

## Changes

- F1 (fixed as proposed): D11 sets `discussion_advisors` to an explicit empty list under `single` or `twill`, reports a lower-layer line for `seats` with no list of its own, and says why per-document migration needs both; the `MigrateConfig` acceptance criterion gains the twill-over-seats and seats-without-list cases.
- F2 (fixed as proposed): D7 makes `discussion_advisors` a guard key, replacing the category-none reach sentences; D8's closing claim now bounds advisors and lens forks alike; D11 drops the local and `run_settings` string-entry cases, noting `ValidateLocal` refuses the key there, and the acceptance criterion follows; Auto-mode assumptions name D7; `support-log.md` Rejected alternatives gains keeping the key category none.
- F3 (fixed as proposed): D3 reads the verdict from the attempt's envelope (`ReadArbiterEnvelope` at `ArbiterEnvelopePath`), shows the cause in parentheses, keeps `running` and `no verdict` for a missing envelope, and shows `verdict unreadable` for an unreadable one; the arbiter acceptance criterion gains a `hand-on (died)` case.
- F4 (fixed as proposed): the unreadable-attempt line sits at the end of the frame, after history-only rows and before D1's state and wait lines, in D3 and in its acceptance criterion.
- F5 (fixed as proposed): D2 places a goto entry as the last line of the round in progress, or before round 1, with the next review visit opening the next round; the goto acceptance criterion renders the goto line at the end of round k.
- F6 (fixed as proposed): D2's hand-on clause now reads "joins the round its output names, opening it only when no earlier entry has", and the hand-on acceptance criterion carries it.
- F7 (fixed as proposed): `continue` became `resume` in D3's rationale and the escalation acceptance criterion.

## Disputed

None.

## Deferred

None.
