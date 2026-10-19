# Fixer-report — merge round 1

## Changes

Both fixes are in commit `214371241`, one commit because the two edits share the `PATTERN.md` headroom budget and neither passes `lyx loom lint-pattern` alone.

- F1 (fixed as proposed): `PATTERN.md:12` now reads "Linking hub-level containers or writing the hub workspace file: from the hub only, never into a worktree."
- F2 (fixed differently): `PATTERN.md:31` now reads "Writing `_launchers` or the workspace file in `fabricengine`: via a hub `os.Root`, never raw `os` or `fslink`."
  - Proposed change not made: the declared text "Writing under `_launchers`, or the hub workspace file, from `fabricengine`: via an `os.Root` at the hub, never raw `os.MkdirAll`, `os.WriteFile` or `fslink`."
  - Why: with F1's declared text, the declared F2 text leaves `PATTERN.md` 47 bytes past the headroom line (`lyx loom lint-pattern`: 15611 bytes against 15564).
  - The review's fallback, shortening another entry the task already touched, has no target: against `main` the task touched only these two entries.
  - The shortened text keeps the declared meaning and matches `writeLaunchers` and `writeHubWorkspace` in `internal/fabricengine/doc.go`.

`lyx loom lint-pattern` is ok after the commit.

## Self-check

- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-code-correctness: not spawned, no code changed.
- fix-affected-set: not spawned, the findings are single-line entries, not sets.

## Disputed

None.

## Deferred

None.
