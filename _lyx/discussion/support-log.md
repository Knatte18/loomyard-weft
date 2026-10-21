# Support log — lyx board rename

## Interview

A darn run holds no interview: the writer read the board entry and the board engine and CLI code, and made every call itself.

## Rejected alternatives

- Building the rename from the existing remove and upsert paths, as the manual route did: it rebuilds the entry from a field map and loses the ID, position and unknown keys, and stamps dates on a new entry.
- Letting the run lock judge dependents untranslated: a run-held dependent's rewritten `depends_on` would read as a scope change and refuse every rename of an entry a running task waits on, though the dependency is unchanged.
- A plain substring match for mentions: a short slug such as `board` would report every entry naming any slug that contains it.
- Rewriting prose mentions in bodies and briefs: the board entry keeps prose untouched, and a blind rewrite could change text that names something else.
- Addressing by `slug` alone: every other addressing verb takes `slug` or `id`.

## Review rounds

_No rounds yet._

## Question ledger

- Should the payload take `id` as well as `slug`? Self-picked yes, matching the other addressing verbs. Resolved.
- Should renaming an entry onto its own slug be a no-op? Self-picked a refusal as an existing target, since it is a mistaken call. Resolved.
- Does the renamed entry count toward `mentions`? Self-picked yes: its own body or brief may still name the old slug. Resolved.
- Should a rename revise the dependents' `revised` dates? Self-picked no: a dependency on the same entry under a new name is no scope change. Resolved.
