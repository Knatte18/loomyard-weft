# Report: what costs Code-Burler-Review its extra minutes

Adds `docs/research/code-review-cost-per-input.md`, a measurement report over 59 archived darn runs
(34 from 2026-11-16 and 2026-11-17, 25 from 2026-11-21 and 2026-11-22).
It changes no code, stencil or config.

## What the report finds

The Code-Burler-Review row grew 386 s to 575 s per run on the mean (300 s to 438 s on the median).
The chair's wait for the assistant reviewer, a Codex seat on every later run, accounts for +174 s of the +189 s.
The row follows the assistant's own run length (correlation 0.98), and the assistant ends after the chair has finished
its own review in all 31 assisted round-1 reviews.
Stopping the assistant's strand after it has ended adds a fixed 15 s per assisted round, +17 s per run.

The suspects named in the entry do not show a cost of their own:

- The tautological-test bullet: the `code-tests` lens fork ran 41 s before and 39 s after.
- The scope lens: 26 s, in parallel, and never the slowest fork.
- The rubric's growth of about 4 KB: the chair's reading time before its first fork went from 89 s to 87 s.
- Mutation survivors: the row rises with them, but so does the diff; the report bounds their share at about 1 min per run
  and does not separate it from Go content.

## Candidate notes the report ends on

- Skip the assistant on a diff of 100 changed lines or fewer: about 31 s per assisted round, 1 LOW kept finding lost of 38.
- Write the lens files before the wait for the assistant: about 40 s per assisted round.
- Stop a finished assistant's strand at once: 15 s per assisted round.
- A time cap on the assistant is a poor trade: the kept findings are not in the short runs.
- A timed replay of survivor diffs would settle the survivors' share; this task's bounds ruled it out.

## Design decisions

- The report reads the chair's and the forks' Claude session transcripts in addition to the archive tags,
  because only they time the chair's wait on the assistant and each lens fork.
  Only timestamps and tool names were read, and the report names this source in its Method section.
- The sample is every qualifying run in the two periods, not about 15 per period, since all were readable.
- Every figure is from the archived records of the periods named; none was timed or run again.

## Merge danger

**Door:** two-way
**Blast radius:** docs
