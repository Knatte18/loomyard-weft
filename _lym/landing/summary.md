# Report: where darn runs lost time since 2026-11-18

Adds `docs/research/darn-slowdown-measure.md`, a measurement of darn run time from the archived run records and gate records.
No code changes.

## Findings

- The doubling does not reproduce.
  Full darn runs took a median of 45.0 min (33 runs, 2026-11-16 and 2026-11-17) and 45.3 min (16 runs, 2026-11-21 and 2026-11-22), with pauses over 30 min left out.
  The entry's 24 and 47 min medians were not found in any cut of the archive.
- Three small changes are real: row gaps +56 s from the records push at each transition, Code-Burler-Review +2.8 min median, and a verify tail of about 75 to 105 s per gate for the reach, redundancy, mutation and red-first steps.
  A shorter test scope per gate and a shorter writer row offset most of them.
- Gate slots, load, hardener live tests and tier wall times are not causes; the Go test cache never hits in a verify gate in either period.
- Sub-second gate failures are test packages that do not compile, which need their rework, and red-first refusals on non-Go diffs that stopped on 2026-11-21.
- The report ends with each cause, its share of a 45 min run and a candidate note for a fix.

## Not done

The second timed run was not made: one run was made at load average 1.5, and its tiers ran from a warm cache.

## Merge danger

**Door:** two-way
**Blast radius:** docs
