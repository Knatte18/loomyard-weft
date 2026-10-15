# Fixer report — plan, round 1

## Changes

- **F1** (fixed as proposed): in card 2, dropped "and false for an empty `files`" from the `symbolFilesCovered` sentence, dropped the Modified-with-neither-side clause from the Modified bullet, and dropped the only-`After` and no-`Before`-or-`After` Modified rows.
  Premise confirmed: quarry v0.2.1's `compareTables` (`internal/engine/delta.go`) sends a key with no before occurrences to `created` and one with no after occurrences to `deleted`, and builds a `ModifiedSymbol` only in its `default` case.
- **F2** (fixed as proposed): in card 2, ended the RefScope clause at "`planparser.RefScope(nil, t)`" and dropped the `plan:nohash` row.
  Premise confirmed: `handle-malformed` in `contracts/specs/loom-plan-spec.md` refuses a handle whose text after `plan:` carries no `#`.
  Card 1's `TestRefMember` keeps its `plan:nohash` row, since it tests `RefMember` directly, not `ScopeGuard`.
- **F3** (fixed as proposed): in card 1, added `plan:internal/planparser#RefMember` as the first bullet of the `**Edit:**` group.

`lyx loom validate-plan` reports `ok: true` after the changes.

## Disputed

None.

## Deferred

None.
