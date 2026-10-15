# Fixer report — plan, round 1

## Changes

- F1 (NIT, consistency), fixed as proposed: in `_lyx/plan/01-engine-siblings.md`, the `TestWriteSiblings` seed sentence now names the one exception, the note N of the note refused row, seeded as a `"kind":"note"` carrying `bug` and given `depends_on` [D] by set-deps.
  The refused-rows bullet names that subject: set-deps giving the note N `depends_on` [D].
  Premise checked against `internal/boardengine/store.go:332`, where `validateWrite` refuses a note with `depends_on` only when the incoming entry is a note.

## Disputed

None.

## Deferred

None.
