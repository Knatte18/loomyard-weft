# Remove the Webster kick-start pack

`kickstart-pack-remove` · Note · enhancement, webster, quarry, loom

## Operator decision 2026-10-17

Remove the Webster kick-start pack, every layout of it (inline, pack file, per card), with its code and config keys.
Glyphs stay in the plan and in the plan gates.

## Evidence

- Four audits, `docs/research/glyph-audit-2026-10-12/`, `-10-13/`, `-10-15/`, `-10-17/`: no layout lowered exploration against no pack (exploration calls per card 5.2 without a pack, 5.8 to 7 with one; calls before the first edit 7 against 4 to 8.5), and the per-card layout costs more per card than the older ones on a different workload.
- A sample of 9 per-card forks (2026-10-17): the forks work file by file and read a file just before editing it; of the spans not read before the first edit, 59% were read later, 23% never read while the file was edited from the fork's own grep, `sed` or build errors, 18% not needed or already covered.
  Several forks grepped for sites the pack had already listed, and none mentioned the pack.
- Webster is about a third of a loom run's active time (`lyx report timing` since 2026-10-10, 105 landed runs: median 30% per run), and reading code is a small part of it.

## Orch recommendation, not a decision

- Keep `lyx quarry resolve` available to the fork, and keep `tools/forkaudit` working for runs without a pack, so a no-pack baseline can be measured.
- Note `enclose-kickstart` builds the pack from tool locations and loses its purpose with this removal.
