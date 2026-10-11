# Batten notices: no attach clause, pair-relative paths, labelled reasons

A batten notice to the orch carried a trailing `; cd <absolute pair path> && lyx reed attach`, absolute paths that repeat the hub prefix in every line, and an `error:` label on every halt reason, awaiting PR-Gates included.
The orch already knows how to attach, and each notice lands in a session that re-reads its context every turn, so this trims the line and makes its wording match what happened.

## What changes

- `renderNotice` in `internal/battenshed/notice.go` no longer appends the attach clause.
- Every path under the task pair in a notice (the condition, the reason and the report clause) is rendered relative to the pair, for example `_lyx/shed/<slug>/drive-reports/drive-report-….md`, since the slug already names the pair.
- The child's reason is labelled by state: `error:` for failed and blocked, `waiting:` for every other state.
- A state-changed notice names what happened, the state and the row it stopped at, for example `board-config-per-worktree: awaiting at PR-Gate; waiting: the pull request awaits review`, instead of `child left running; child state awaiting, error: …`.
  The driver-busy notice leads with the same text.
- `InnerRunDeps.AttachDir` is renamed `PairDir`: it now supplies the base directory for relative paths, not an attach target.
  The wiring in `internal/battencli/wire.go`, the nil-legal list in `internal/testkit/envkit/envkit.go` and the tests follow the rename.
- `internal/battenshed/doc.go` describes the new notice wording.

## Design decisions

- `pairRelative` strips the pair prefix only where it starts a path, not wherever the string occurs, so a longer path that merely contains the pair directory is left whole.
- `Status.Error` is unchanged in shed; only the notice's label depends on state, because shed keeps every halt reason in that field.
- A `PairDir` that cannot be resolved skips the notice with a warning, as `AttachDir` did.

## Checked by hand

Nothing outstanding; the notice tests and the battencli lifecycle integration test cover the new lines.
