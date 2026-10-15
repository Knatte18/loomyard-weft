# Webster master reads no code, only the plan

`webster-master-no-code` · Note · bug, webster, stencil

## Seen

`contracts/stencils/webster/webster-template-master.md` tells the master to read the codebase structure and conventions and the plan overview once, up front, so that every implementer fork inherits what it read with no cold orientation.
The 2026-10-12 and 2026-10-13 pack audits (`docs/research/glyph-audit-2026-10-12`, `-10-13`) found forks explore only about 6 calls before their first edit, because they inherit that orientation.

## Operator decision (2026-10-15)

The master must not read code.
Webster rewrites the code, so what the master read goes stale for each fork after the first batch, and every fork inherits those stale reads.
The master reads only the plan and drives the loop; each fork reads the code its own card needs, fresh, when the card starts.

## Idea

Rewrite the master orientation to the plan only (`00-overview.md`, the Card Index, Shared Decisions) and drop the inherit-my-reads framing from the master template and the fork prefix.
The per-card pack of `kickstart-per-card` then becomes the fork read set rather than an extra on top of inherited reads.
Re-measure fork exploration with `tools/forkaudit` afterwards: forks will explore more by construction, so compare total reads and stale-read edits, not calls before the first edit.