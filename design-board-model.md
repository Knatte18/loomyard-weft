# Board as one list: tasks and notes with labels, GitHub issues pulled in

`board-model` · Tasks · feature · done

## Goal

The board becomes the operator's one list: two layers, tasks and notes, both labelled, with GitHub issues pulled in as notes so nobody tracks GitHub and the board side by side.
GitHub stays the inbox, because `lyx selfreport` files from any repo lyx runs in and only GitHub is reachable from all of them; the board is loomyard's plan.

## Decided (operator, 2026-10-04)

- Two layers: a `kind` field, `task` (concrete, runnable work) or `note` (an idea), replaces `tier`; tier 2 and "Next up" are gone from the model, the `promote` verb's vocabulary and the README.
  `promote` becomes changing `kind` from `note` to `task`.
- Labels are a field on both tasks and notes; the bracket prefixes in today's briefs (`[orch][ide] …`) migrate into it.
  `lyx board list` and `find` filter by label, and the README shows labels.
- `type` (feature, bug, chore, design) folds into labels, so there is one classification, on two axes:
  - Type: `bug` or `enhancement`. A note carries exactly one; a task carries the type labels of the notes it bundles, so it may carry both.
    `chore` folds into `enhancement`; `design` is not a type but the maturity label `undecided` (what to build is not yet decided), removed once it is.
  - Area: any number, e.g. `orch`, `webster`, `board`, `ide`.
- The type and label lists live in config, so the operator changes them with one edit; a label or type outside the lists is refused.
- The README and `lyx board list` group notes by type (Bugs, Enhancements), and filter by area.
- `depends_on` belongs to tasks only; notes have none, and a relation between notes is a sentence in the body (`card-batching`'s dependency on `plan-glyph-precision` migrates that way).
- `lyx board upsert` reads the body from a file or stdin, so editing a body never means splicing JSON in a shell one-liner.
- An intake verb pulls GitHub issues in: it lists open issues not yet on the board, imports chosen ones as notes (labels carried over, a link back, the issue number recorded so an issue is never imported twice) and closes the issue with a pointer to its note, and closes noise with a stated reason.
  The verb is mechanical; the orchestrator makes the triage calls.

## Open, for Discussion

- Migration of existing entries: tier 1 → `task`, tier 2 and 3 → `note`; brief prefixes → area labels; `type` → `bug`/`enhancement`, with `design` entries also getting `undecided`.
- Which repo the intake verb reads (the selfreport target) and how it authenticates (selfreport's token resolution).

## From `board-labels`

Seen 2026-10-03: the board model is now tasks (tier 1) and notes (tier 3, one per entry), with labels as a bracketed prefix in each note's brief (`[ide][orch] …`) because the board has no labels field; tier 2 is unused, and README rendering already says Tasks and Notes (`6a74ee1be`).

## From `board-body-file`

Seen 2026-10-03: `lyx board upsert` takes the whole body inside one JSON argument, so changing one section of an entry means reading the body, splicing it and re-encoding JSON in a shell one-liner, which breaks on quoting (an apostrophe in the text) and pushed the orch into a throwaway script.

## From `issue-intake`

Since filing has no approval gate (operator, 2026-10-03, see `agent-self-reflect`), filtering happens when issues are pulled back in.
Idea (operator): a `lyx` command rather than a skill, that ingests open GitHub issues, drops noise and duplicates, and proposes board entries or folds into existing ones.
See `manifest/designs/curation-triage.md` (moved to `internal/boardengine`'s doc by `pattern-init`).
The first intake was done by hand on 2026-10-04: the open issues became notes and were closed with a pointer.
