Task: check the `_lym/landing/summary.md` Door edit for agreement with its surroundings (fix-consistency lens).

**Finding 1: LOW.** The changed text is the `**Door:** one-way` line in `_lym/landing/summary.md`, under `## Merge danger`.
- The fix adds a prose sentence between the Door line and the Blast radius line, and the spec's example puts the two lines adjacent (`contracts/specs/loom-final-summary-spec.md:34-35`).
- The spec says only that the section runs to the next level-one or level-two heading and that a door "other than those two words" is refused (lines 44, 47-48).
- It does not say the lines must be adjacent, so I expect the parser to accept the edit, though I did not read `internal/summaryparser` to confirm.
- The risk is that the parser reads the Door value to the end of the line (`one-way`), which the edit leaves intact, so I found no concrete breakage.
- The blank line the edit adds before `**Blast radius:**` is harmless.
- The edit agrees with the spec's rule at line 39 (a record format another binary reads is one-way) and with the review's fix text. Nothing else in the landing summary cites the door.

No further findings.

Tool calls:
1. `grep` on `contracts/specs/loom-final-summary-spec.md` for the door rules and example format, which my context lacked.
