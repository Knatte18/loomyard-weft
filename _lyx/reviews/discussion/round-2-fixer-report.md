# Round 2 fixer report

## Changes

- F1, MEDIUM, fixed as proposed.
  Premise checked: `git ls-files` lists tracked non-Go files spelling `_lyx/`, among them `contracts/recipes/loom-recipe.yaml`, `contracts/recipes/darn-recipe.yaml` and specs under `contracts/specs/`.
  `decision-record.md` Constraints, first bullet: the amendment now admits a shell tool under `tools/` that cannot import `internal/lyxdirs`, provided it spells each literal once in a variable block at the script's head.
  `PATTERN.md` stays in Scope In.
  `support-log.md` Round 2 entry records the premise check.
- F2, LOW, fixed as proposed.
  `decision-record.md` D1 states that `--out` names a file: an existing directory there exits 1 naming `--out`, writing nothing, and an existing file is replaced.
  The unwritable-`--out` acceptance criterion is extended with the existing-directory case.

## Disputed

None.

## Deferred

None.
