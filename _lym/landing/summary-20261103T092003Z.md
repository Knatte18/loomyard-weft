# Drift-Baseline halt carries the capture child's error

A capture child that ends before it records its start now puts its own error in the halt, and an error a fresh start cannot change no longer offers `lym loom resume` as the way forward.

Closes #26

## What changes

- **The halt carries the child's error.**
  `startFailure` in `internal/driftengine/spawn.go` reads the child's log for the last JSON error envelope (`{"ok":false,"error":...}`), the shape every failing `lym` command writes.
  It appends that error to the message, folded onto one line and cut to 400 runes with an ellipsis.
  This applies both when the child ends early and when it stalls past the record bound, in the Drift-Baseline row's form and in the `lym drift capture` form.
- **The child marks an error a fresh start cannot change.**
  `lym drift capture-child` sets `lasting_fault: true` (`driftengine.LastingFaultKey`) in its error envelope when its worktree does not resolve from its cwd: the cwd and anchor check, an invalid or stale anchor marker, or no git repository.
  A fresh start runs in the same anchored directory and resolves it the same way, so it fails the same way.
- **A lasting error drops the resume.**
  For such an error the Drift-Baseline halt says `lym loom resume` cannot change it.
  Its ways forward are `lym drift defer` followed by `lym loom resume`, or a fix of the error's cause.
  The `lym drift capture` refusal does the same: it drops `run lym drift capture again`.
  A child error without the mark keeps the existing ways forward.

## Design decisions

- The child classifies its own error, because it holds the error value and can test it with `errors.Is`.
  The parent only reads a flag from the envelope, so `driftengine` matches no error text and still imports no `lymcwd`, as `PATTERN-cwd-resolution` requires.
- Only a failure to resolve the worktree is marked lasting.
  The child's later failures, such as a config load, are left unmarked: the row checks the config before it starts the child, and an unmarked error keeps the ways forward it had before.
- The lasting halt still names `lym loom resume` after `lym drift defer`, since a deferral moves the run on only through a resume.
  What it drops is the resume offered as a capture retry.
- A child that writes no error envelope, such as one that crashed, leaves the halt naming its log alone, as before.
  Its log's last line would be a stack frame or a logger line, which says less than the path.

## To check by hand

- Start a Drift-Baseline capture child in a directory other than the anchor, for example by moving `.lym-anchor` after a run has started.
  Confirm the halt in `lym loom status` quotes the cwd refusal, says a resume cannot change it, and names only `lym drift defer` and a fix.
