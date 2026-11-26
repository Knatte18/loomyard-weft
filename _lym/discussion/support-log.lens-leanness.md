```yaml
lens: leanness
outcome: reported
```

Task: a leanness review of the decision record and support log, looking for content to move or cut.

**MEDIUM: Notes for the plan writer, third bullet.** A rule is hidden in the optional notes. "A load failure skips the check for that tick with one Warn" is required behaviour for the gate-hold watch. The plan writer may treat notes as optional hints. By the completeness-before-leanness test, move that sentence into D6, ideally with an acceptance bullet. Leave only the wiring hint, that `internal/beamcli` loads the strict gate config, in the note.

**MEDIUM: Constraints, last bullet (#207 fixed by `77630a4f1`).** This is deliberation history, not a constraint. It sets no requirement the plan writer must meet. The support log already holds it in Turn 1 and Q1. The part that matters, that a batten halt outside Run-Shed derives as running or landing, is already in D1's rationale. Cut the bullet from the record.

**LOW: Scope Out, first bullet, against D8.** The bullet lists the same five notices that D8 lists, and both say they keep their behaviour. Keep one and point the other at it, e.g. "Out: changes to existing notices (D8)".

**LOW: Constraints against the decisions' rationale.** Three constraint bullets restate what the decisions already say:
- `PATTERN-run-fsm` is in D1's rationale.
- `PATTERN-config-layers` is in D2's text.
- `PATTERN-child-processes` (spawns nothing) is in D6's bound.

Keep each fact in one place. The constraint bullets that add something are `PATTERN-sole-parsers`, `PATTERN-daemon-wakeups`, `PATTERN-tolerant-records` and `PATTERN-owner-machine`.

**LOW: Open risks, second bullet (T4).** It concludes there is no shared code and no conflict, so it names no risk. It belongs in the support log's Turn 3, or can go entirely. Scope Out already names T4's ownership of the kill.

**LOW: Open risks, first bullet.** The double mail is a cost the record has already accepted through D7, D8 and support-log Q3, not an open risk. Either shorten it to the residual risk (mail volume) or move it to the support log.

**LOW: Goal bullets against D2, D3 and D6.** The three Goal bullets list the stop states, the gate bound and the rows level again. Keep only the outcome (the parent hears every stop and every over-long slot hold, with no Monitor) and let the decisions carry the lists.

**LOW: Auto-mode assumptions.** The section repeats values the decisions already state: the `stops` default, the `[run watch]` prefix, 1.5 and 90. The section is required, so keep it, but one line each pointing at D2, D3 and D6 would be enough.

**LOW (outside lens): D2, last bullet, is self-contradictory.** It says a bad value "refuses the dobby's lookup with … way forward" and also that the run "reads as `stops`". A refusal that is then ignored needs one wording, e.g. "is logged once with the key's three values and reads as `stops`".

I made no further tool calls.
