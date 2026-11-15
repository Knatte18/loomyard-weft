# Map the state lym keeps, where it lives and whether git holds it

Adds a read-only study, `docs/research/loom/lym-state-map-2026-11-15.md`, and its line in `docs/research/README.md`.
It is the groundwork for the Motif `adopt-redesign`; it names facts and sources and proposes no design.
No code changes.

The study holds:

- An inventory of the state lym keeps for a hub and its runs: the file or place, the code that writes and reads it,
  whether git holds it and on which branch or repository, and how it is pushed.
  For state git does not hold, it says what breaks when another machine lacks it and whether lym can rebuild it from pushed state.
- The eight steps `lym adopt` takes since the 2026-11-11 rework, each with the state it reads and writes,
  its git operations, and which of them can fail when the old owner's machine is off or holds unpushed work.
- The adopt failures on record, from `_bolt/.lym/adopt/`, the hibernate and orch watcher logs and the board entries the Motif bundles.
- A first-draft state machine of a run as batten, loom, dobby, tender and the board record it, with the places their vocabularies disagree.

## Design decisions

- The report records one run at one point in time, as the other studies under `docs/research/` do, so it is not rewritten when the code changes.
- Claims the study could not confirm in code or on disk are marked UNVERIFIED and collected in its last section.
- The board body of `adopt-redesign` says no adopt error text was recorded and the last adopt was on 2026-11-11.
  The adopt records on this machine contradict both: `adopt.out` holds error text, and `adopt.json` is from 2026-11-15 and ends green.
  The report states this and leaves the board entry alone.
- The study ran no adopt, no gate test and no run, so a fix commit's effect on a recorded failure is judged from its title only.
