# Decision record — lyx board rename

## Goal

Add `lyx board rename {slug|id, to}`, which changes one entry's slug in one atomic board write, keeps every other field of the entry, rewrites the old slug in every other entry's `depends_on`, and reports the entries whose body or brief still mention the old slug.

## Scope

- In: a store-level rename in `internal/boardengine`, a `Board.Rename` facade method under the write lock, the `rename` verb in `internal/boardcli`, its help, payload declaration and tests, and the docs that name the board's verbs (`boardcli` and `boardengine` `doc.go`, the `ly:board` skill).
- Out: rewriting prose mentions of the old slug, renaming worktrees, branches or run records keyed by the slug, and renaming a design doc by hand (the render after the write drops the old design file and writes the new one).

## Decisions

### The renamed entry keeps every field but its slug

Decision: the rename replaces only `Slug` on the stored entry; ID, position in board.json, body, kind, labels, priority, issues, dates, run settings, profile, status and kept unknown keys stay as stored.
Rationale: the board entry asks for a rename that loses nothing, and the existing upsert and merge route is what dropped fields.

### Dependents are rewritten, and a rename is no revision

Decision: every other entry's `depends_on` has the old slug replaced by the new one in place, keeping its order.
The critical section judges dates and the run lock against the loaded entries with the same rename applied, so neither the renamed entry nor a rewritten dependent gets a new `revised` date or a run-lock refusal from the rewrite alone.
Rationale: a dependency on the same entry under a new name is the same scope, so it is no revision and changes no run-held dependent's scope.
Bound: the translation applies only the one old-to-new slug pair the rename names, and only for the rename write; any other change in the same write is still judged in full.

### Refusals

Decision: the rename refuses, writing nothing, when the source entry is absent, when the target slug is empty, longer than `boardengine.MaxSlugLength`, or already names an entry (the source itself included), and when the source entry has the run-status form.
The run-held refusal is the existing `*RunLockedError`, so it matches `ErrRunLocked` and names the same ways forward.
Rationale: the board entry names these refusals; reusing the run-lock error keeps one refusal text for every scope change on a held entry.

### Payload and output

Decision: the payload is `{slug|id, to}`, the addressing pair every other addressing verb takes plus a required `to` string.
The success envelope is `{"ok":true,"task":{…},"mentions":[…]}`: the renamed entry and the slugs of the entries whose body or brief mention the old slug after the write, in store order, the renamed entry included, and `[]` when none does.
Rationale: slug-or-id addressing matches the other addressing verbs; an always-present `mentions` list lets a caller act on it without a presence check.

### A mention is a whole-slug occurrence

Decision: an occurrence of the old slug counts as a mention only when the characters on both sides of it are not letters, digits, `-` or `_`.
Rationale: a plain substring match would report `board` inside `board-rename` and flood the list for a short slug.

## Constraints

- The board is read and written only through `lyx board`; the verb goes through `boardCriticalSection`, so it saves, renders and syncs like every other write.
- `PATTERN-refusal-way-forward`: every new refusal names the way forward.
- `PATTERN-cli-cobra`: the verb is a cobra subcommand of the `board` tree.
- The verb's `Use` token must match its `payloadKeys` declaration, which `TestPayloadUse_MatchesDeclaredKeys` checks, and its help states the slug limit.
- `PATTERN-test-economy` and `PATTERN-test-speed`: tests extend the existing tables where one covers the behavior, and stay at the unit tier.

## Auto-mode assumptions

- The payload addresses by `slug` or `id`, though the board entry names only `slug`.
- A rename of an entry onto its own slug is refused as an existing target rather than treated as a no-op.
- The renamed entry's own body or brief counts toward `mentions`.

## Open risks

- Run records, worktrees and batten seeds keyed by the old slug are not renamed; the run-held refusal keeps a live run's entry from being renamed, but a finished or abandoned run's records keep the old slug.

## Acceptance criteria

- A rename keeps the entry's ID, position, body, kind, labels, priority, issues, created and revised dates, run settings, profile, status and unknown keys, and changes only its slug.
- Every other entry's `depends_on` names the new slug in place of the old one, and their `revised` dates are unchanged.
- A rename refuses, with nothing written, an absent source, a target that exists, an empty target, a target over the slug length limit, and a run-held source (matching `ErrRunLocked`).
- A rename succeeds when a run-held entry depends on the renamed one, rewriting that dependent's `depends_on`.
- `mentions` lists the entries whose body or brief mention the old slug as a whole slug, `[]` when none does, and not an entry whose text holds the old slug only inside a longer slug.
- The README and design docs render under the new slug, and the old slug's design file is gone.
- `lyx board rename --help` names `slug`, `id`, `to` and `mentions` and states the slug limit; its `Use` token matches its declared payload keys.
