# lint-comments --base lints the working tree, not committed HEAD

`lym loom lint-comments --base <commit>` compared `<commit>` with committed HEAD.
A fix to a flagged comment showed up in the lint only after it was committed, which forced a commit per check cycle.

Now `--base` compares the working tree with `<commit>`, uncommitted edits and untracked `.go` files included.
Without `--base` the verb still compares the working tree with HEAD.
The verb's help and `docs/overview.md` describe this.

Closes #182

## Design decisions

- `commentlint.Lint(worktree, base)` always reads the working tree; an empty base means HEAD.
- The committed-range reading moves to a new `commentlint.LintCommitRange(worktree, base)`, which diffs base against HEAD and reads no working tree.
  The verify gate in `internal/loomshed` (`NewVerifyGate`) calls it, so the gate keeps verifying the committed tree and an untracked scratch file cannot fail a round.
  `PATTERN-gate-slot.md` records the split.
- An untracked file that the base already holds, such as one removed from the index in a later commit, uses its own path as the base side.
  Only the lines it changed count as new, so its legacy wrapped comments are not flagged.
  Finding that out takes one `git cat-file -e` call per untracked `.go` file, marked as a raw git site and pinned in `cmd/lym/checkedcall_test.go`.
- Untracked files that are not `.go` files are no longer listed.
- `internal/loomcli` and `internal/gatecli` callers pass the same arguments as before; only `--base` changes meaning.

## To check by hand

- `lym loom lint-comments --base <commit>` on a tree with an uncommitted wrapped comment reports it.

## Merge danger

**Door:** two-way
**Blast radius:** commentlint
