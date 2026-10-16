# Webster renames through a type-aware tool, the edit rule's one exception

`webster-rename-script` · Note · bug, stencil, webster, burler

## Seen

Found by the stencil intent audit (`docs/research/stencil-intent-audit-2026-10-15/findings.md`, F7); line numbers as of `main` on 2026-10-16.

- `contracts/stencils/edit/edit-directive.md:6` bans every script that rewrites files, `python` and `node` heredocs included.
- `contracts/specs/loom-plan-spec.md:125` and `:334` make a `Rename` of a symbol an AST-aware script plus a grep, never a text or regex replace; for Go the named library is `go/ast` with `go/types`.
- `contracts/stencils/loom/loom-rubric-webster-review.md:79` asks the reviewer to confirm that script ran and passed, which a diff cannot show.
- The webster fork prompt (`webster-body-implementer.md`) never tells a fork to run such a script, so a fork obeys the edit directive and renames by hand.

## Decision

Operator decision (2026-10-16): the AST-aware rename script is required for a `Rename` in Webster and stays.
It is the one exception to the edit directive, and only for this part of Webster.
Python is acceptable for that script when the Go-native route is not available, since a type-aware Go tool may not be installed on a loomyard user's machine.

## Idea

Agent's suggestions:

- The edit directive names the exception once: a fork working a `Rename` card may run a type-aware rename tool or script, and nothing else may.
- The fork prompt tells a fork on a `Rename` card how: `gopls rename -w <file>:<line>:<col> <new>` when `gopls` is on `PATH` (locate with `lyx quarry resolve`), else a throwaway program under `.scratch/` on `go/ast` and `go/types` run with `go run`, which needs only the Go toolchain a Go repository already has; then the grep pass.
  Python stays the operator-sanctioned last resort.
- Webster-Review checks the rename's outcome, which a diff shows: no reference to the old name left and the symbol's doc comment updated, instead of "the script ran".
- The spec's Rename row and quarry section are updated to match.
