# Drift-Baseline halt shows the capture child's error

A capture child that ends before it records its start now puts its own error in the Drift-Baseline halt, so the parent no longer has to open the child log to learn the cause.
When the child marks that error as one a fresh start cannot change, the halt drops `lym loom resume` as a way forward and names only `lym drift defer` and a fix of the cause.

Before, the halt read "drift: the capture child ended before it recorded its start" with the log's path, and advised `lym loom resume`.
A cwd refused by the anchor check recurred on every entry, so the resume could not help, and the cause sat only in the log under `_lym/drift/logs/`.

Closes #26

## What changes

- `internal/driftengine/spawn.go`: `startFailure` reads the last JSON error envelope (`"ok": false`) in the child log and appends its `error` text to the message.
  The text is folded onto one line and cut to 400 runes, with `…` marking a cut.
  A log with no error envelope, such as a crashed child's, or one that cannot be read, leaves the message as before: the log's path alone.
- `LastingFaultKey` (`lasting_fault`) is a new key in the child's error envelope.
  `childNoRecordError` carries it as `lasting`.
- `internal/driftengine/baseline.go`: `captureAgain` turns a lasting `childNoRecordError` into a Stuck that says `lym loom resume` cannot change the error and names `lym drift defer` followed by `lym loom resume`, or a fix of the cause.
  Other no-record errors keep the resume-to-capture-again halt, now with the child's error in it.
- `internal/driftcli/cli.go`: `runCaptureChild` marks its error envelope lasting when `openWorktree` fails with `lymcwd.ErrNotAGitRepo` or `lymcwd.ErrCwdOutsideAnchor`.
- The same applies to the CLI's own `lym drift capture` start (the `FallbackDetach` form): it no longer offers "run lym drift capture again" for a lasting error.
- Package docs in `driftengine` and `driftcli` describe the new behaviour.

## Design decisions

- The child marks lasting faults itself, in its envelope, instead of the parent matching error text.
  The parent has no reliable way to tell from a string whether a restart would end the same.
- Only the two cwd refusals are lasting.
  A read error on the anchor marker may pass on a fresh start, so it stays resumable.
- "Last error envelope wins": the child may log several, and the final one is what ended it.

## Tests

Added in `spawn_test.go`, `baseline_test.go` and `anchor_integration_test.go`:

- the error carried on one line, last-envelope selection, skipped `ok` envelopes, and the rune-based cut;
- lasting versus resumable, with and without `FallbackDetach`, and the halt dropping the resume offer;
- an integration check that `lym drift capture-child` run at the worktree root of an anchored hub is refused and marked lasting.

Not run by hand: a full Drift-Baseline halt in a live run against a real cwd fault.
