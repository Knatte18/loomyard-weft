```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set self-check of the F1/F3 stencil edits.

Findings left unchecked: none. F2 is a `decision`-class finding that the fixer does not own.

no findings

- F1: the search for the measurement rule's no-parent branch finds it only in `darn-fragment-make-change.md` and `darn-fragment-make-change-gated.md`, and both carry the new closed-scope clause. The other hits are headers and doc comments that only say the writer asks first, which still holds.
- F3: no "after which you" or "which you run/then" remains outside `_lym`, so both named sites are covered and there is no third.

Tool calls:
- `git grep` with the pathspec `:!_lym`, issued twice in one message: both failed on pathspec syntax and read nothing.
- `git grep` with `:(exclude)_lym` for the measurement-rule and clause phrases: my context had no repository-wide search for them.
