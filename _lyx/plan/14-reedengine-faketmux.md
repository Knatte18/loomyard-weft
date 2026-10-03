# Card 14 — reedengine-faketmux

**Create:**
- `reedengine/faketmux_test.go`

**Edit:**
- `reedengine/apply_test.go`
- `reedengine/attach_test.go`
- `reedengine/reapply_test.go`
- `reedengine/watchloop_test.go`

**Uses:**
- `reedengine/lock_test.go`

**Intent:**
Replace reedengine's hand-written `execHook` closures and bespoke recorders with one scripted tmux fake (#325 §3), starting with the files that carry the bespoke recorders.

`faketmux_test.go` (`package reedengine`, untagged) defines `fakeTmux`, installed on an engine's `execHook`:

- per-verb scripted answers and errors keyed by tmux verb (`args[0]`), with optional disambiguation of `display-message` by its `-F` format string;
- a mutex-guarded call log with `Calls()`, `Sequence()` (verbs in order) and `ArgvFor(verb)`;
- a `mustNotCall(verbs...)` guard that fails the test when a listed verb runs;
- an unscripted verb answers empty with no error, matching what most closures do today.

Convert `apply_test.go`, `attach_test.go`, `reapply_test.go` and `watchloop_test.go` to it, deleting `attachScript`/`attachRecorder`/`newAttachHook`, `applyHookRecorder`/`newApplyRecordingHook`, `driverHook` and `scriptedHook` (and `watchloop_test.go`'s copy of it).
Every scripted answer keeps its exact value; a test asserting call order asserts on the fake's log.

Delete the re-sets of `e.cfg.Width, e.cfg.Height = 100, 21` and `CollapsedRows, MinFullRows = 2, 3` in these files: they equal `newTestEngine`'s defaults in `lock_test.go`.
A re-set that sits in a test whose subject is that layout keeps the value as an explicit `defaultLayoutCfg`-style constant rather than a silent duplicate.

Apply D10 to every touched test.

**ImpactSummary:** The untagged reedengine test files carrying bespoke recorders move onto one shared in-package fake; scripted values and call-order assertions are preserved.
