# Webster master reads only what every fork shares, never source code

`webster-master-no-code` · Task · bug, webster, stencil · running · Webster

## Seen

`contracts/stencils/webster/webster-template-master.md` tells the master to read the codebase structure and conventions and the plan overview once, up front, so that every implementer fork inherits what it read with no cold orientation.
The 2026-10-12 and 2026-10-13 pack audits (`docs/research/glyph-audit-2026-10-12`, `-10-13`) found forks explore only about 6 calls before their first edit, because they inherit that orientation.

## Operator decision (2026-10-15)

A serious bug: Webster was never meant to work this way.
The master reads only what is common to every fork and stable during the run: the plan, `CLAUDE.md`, `PATTERN.md` and the like.
It reads no source code: Webster rewrites the code, so nothing else is stable during the run, and every fork after the first would inherit stale reads.
Each fork reads the code its own card needs, fresh, when the card starts.
The up-front codebase read has stood in the master template since at least 2026-07-27 (`d89a56232`).

## Idea

Rewrite the master orientation to the plan only (`00-overview.md`, the Card Index, Shared Decisions) and drop the inherit-my-reads framing from the master template and the fork prefix.
The per-card pack of `kickstart-per-card` then becomes the fork read set rather than an extra on top of inherited reads.
Re-measure fork exploration with `tools/forkaudit` afterwards: forks will explore more by construction, so compare total reads and stale-read edits, not calls before the first edit.