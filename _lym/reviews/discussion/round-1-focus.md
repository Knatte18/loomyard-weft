---
round: 1
focus:
  - "D10 and D13, attack surface: D10 drops the seed-freshness check and the stat race guard, and D13 removes the per-recipe arbiter and tackler override and makes a stale profile key a start-time refusal. Check that each states what it can now let through and what bounds it, and that D10's claim about which `CheckSeed` verdicts Shed pre-empts is the stated evidence for dropping the rest."
  - "D4 and D6, halts and overwrites: Record-Write's no-overwrite rule and the Darn-Write no-commit halt are the new routing edges. Check that each Decision says what the halt or the done-on-existing-record path can skip, and that a darn task whose right outcome is no code change is bounded."
  - "D2 and D8, the pawl exception: D2 admits pawl alone to `lym loom`, while D8 keeps pawl's verify source and branches. Check that the exception is bounded in every Decision it touches (D3 verb refusals, D7 snapshot, D8 wiring) and that none of them lets another non-loom recipe through by name."
  - "Open risks: the line on `batten.yaml` asks the parent to confirm what 'in shed.yaml alone' covered. Decide whether that is a question only the parent can settle (class `decision`) or a choice the writer could make, and whether the other risk lines are settled by a recorded parent decision."
  - "D5 against the discussion spec: D5 runs the history check on tapestry records, while the spec says a tapestry run's discussion check reads no history for `Added after Discussion` entries and PATTERN bullets. Check that the record names the spec change as part of the task, so the two do not contradict after landing."
  - "Relocation: look for deliberation narrative in `decision-record.md` that belongs in `support-log.md`, such as rationale restating a rejected alternative, applying the completeness-before-leanness test before any move."
---

Seed pass over the record pair; the focus entries name where to look and leave severity to the reviewer.
No entry targets a missing optional subsection, a rejected alternative absent from the record, or call-site enumeration.
