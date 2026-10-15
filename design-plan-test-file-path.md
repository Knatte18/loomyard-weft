# Plan cards name added tests by Create handle, never a plain test-file path

`plan-test-file-path` · Note · enhancement, loom, stencil

## Seen

`contracts/stencils/loom/loom-template-plan.md` lets a card's `**Edit:**` list a plain test-file path "for a test file the card adds tests to without changing an existing member".
The same stencil already makes each new test function in an existing file a `**Create:**` declaration with a `plan:` handle, so the plain path only repeats where those new tests go.
No recorded reason for keeping both was found; the rule came in with `78a2b1f14` and was tightened in `c279a38ba`.
The 2026-10-15 pack audit (`.scratch/glyph-audit-3/`, to be committed under `docs/research/`) found that edits to files with no pack span were mostly md, yaml and test files, which this exception and the non-Go exception allow.

## Operator view, not a decision (2026-10-15)

Test files should not be an exception to glyph references.

## Idea

- Drop the plain test-file path from `**Edit:**`: a card that adds tests to an existing file lists them only as `**Create:**` declarations with `plan:` handles, and the declaration carries the file.
- Have the plan gate refuse a plain `_test.go` path under `**Edit:**`, naming the way forward, and update `loom-plan-spec.md` and the stencil together.
- Check that Webster's kick-start pack and the fork prompt still tell the fork which file the new tests go in.
