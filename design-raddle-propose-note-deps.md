# raddle-propose passes a dependency on a note that Raddle-File refuses

`raddle-propose-note-deps` · Note · bug, shed, board

## Seen (2026-11-11, raddle of gate-scope-by-coverage-map)

The raddle proposal gave part `testmap-cross-package` a `depends_on` on the note `testmap-upkeep`.
`raddle-propose` validated the proposal and passed it, and the operator was asked to approve it.
Raddle-File then refused the entry because a note cannot be depended on, and the run failed at Raddle-File.
The refusal text ends in "promote \"testmap-upkeep\" with ;", so the command it names is missing.

## Idea

Make `raddle-propose` refuse a `depends_on` that names a note, with the way forward (promote the note, or drop the dependency), so the check runs before the operator approves.
Complete the Raddle-File refusal text with the command it means.
