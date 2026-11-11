# A beam orch passes work outside its beam to its parent orch

A beam's orch still writes the board only under its own beam, and nothing lets it write outside.
Work that belongs outside the beam now has a route that spares the operator a second order:
the beam's orch sends the wish to its beam's parent orch through creel, and that orch places the work.

## Changes

- `contracts/orchrole/orch-role.md`, under `## The board`, gains the rule.
  A beam's orch sends the wish, what to make and why, with `lym creel post <parent orch> --expect-reply "<wish>"`.
  The parent orch is `<shortname>:<parent_beam>:orch`, with `parent_beam` from `lym board get <beam>`, or `<shortname>:orch` for the prime.
  The parent decides the placement from its running tasks and their overlap, `depends_on`, whether the work is a project of its own, and its run count.
  It may place the work differently and tells the operator of a collision.
  A nested beam's orch passes on a wish that belongs higher up to its own parent the same way.
- `internal/boardcli/runsettings.go`: the board's run_settings and profile guard, which refuses a beam's orch an entry outside its beam,
  now names that route as its way forward.
  The parent orch is read through `boardengine.Board.ParentBeamOf` on the beam, so a nested beam's orch is pointed at its parent beam's orch.
  Other sessions with a slug keep the old refusal, which names the operator, the hub orch or the orch of the entry's beam.
- `internal/boardcli/doc.go` documents the new way forward.

## Design decisions

- The new way forward covers the one Go refusal that stops a beam's orch at its beam's edge, the run_settings and profile guard.
  This guard is what refused the root project in the reported case, because new entries carry a profile.
- The rule sits under `## The board` rather than `## Messaging` or `## Working rules`.
  The rule is about who writes which part of the board, and it sits beside the rule on a beam project's scope changes, which also passes a call to another orch.
- `lym creel post --parent` is not used: from a beam it resolves to the beam's own orch, so the role and the refusal name the parent orch by its strand name.

## Tests

In `internal/boardcli/verbs_test.go`, the `stepRunSettings` subtest on a beam's orch changing only its own beam's entries
now checks each refusal's way forward.
A beam's orch is pointed at `ly:orch`, a nested beam's orch at `ly:bm:orch`, and any other session gets the old refusal.
