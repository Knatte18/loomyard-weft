# Plan gate does not catch declared PATTERN.md growth that exceeds the headroom line

`plan-gate-pattern-headroom` · Note · enhancement

Imported from [issue #735](https://github.com/Knatte18/loomyard/issues/735).

A plan-stage burler round (plan, round 2) had to measure by hand whether the plan's declared `PATTERN.md` growth fit the headroom line.
The round's finding asked for a named cut of at least 370 bytes across `PATTERN.md` entries outside the three the plan declares, keeping every rule, marker and link.
Measuring each candidate gave wording-only cuts of 152 bytes, and the overview's entries are already terse, so the proposed fix could not be carried out.

This is abnormal because the plan declares more `PATTERN.md` growth than the headroom line admits, and neither Discussion nor the plan gate catches that before a burler round.
The check is a mechanical byte comparison, yet it fell to a reviewing agent to measure it manually, costing a round and leaving an unsatisfiable finding.

Suggested fix: make the plan gate (or the Discussion stage) compare the plan's declared `PATTERN.md` growth against the current headroom and refuse or flag a plan that exceeds it, so the plan is corrected before review.

Provenance: task `batten-start-speed`, from note `burler-plan-r2.md`.
