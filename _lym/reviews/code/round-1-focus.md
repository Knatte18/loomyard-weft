---
round: 1
focus:
  - "internal/driftengine/baseline.go captureAgain and spawn.go startFailure: the board entry says a lasting error's halt drops `lym loom resume` as a way forward and names only `lym drift defer` and the fix of the cause; the new text names `lym drift defer followed by lym loom resume`. Settle whether that still honors the entry, reading `lym drift defer` as the step that makes the later resume succeed."
  - "internal/driftcli/cli.go runCaptureChild: only the `openWorktree` failure is marked `lasting_fault`. Settle whether every error `openWorktree` returns is one a fresh start in the same cwd cannot change, or whether a transient cause is wrongly told that resume is useless; and whether other cwd or anchor-check refusals the entry names reach the same marking."
  - "internal/driftengine/spawn.go readChildReport and boundChildError: settle the last-error-line extraction against the entry (last error line, bounded). Check a log that holds a non-envelope tail, a multi-line or very long envelope line (bufio-free split on `\\n`), a non-string `error`, and rune-safe cutting at `childErrorMaxRunes`."
  - "Scope against the entry: the diff also changes the `lym drift capture` (FallbackDetach) message and adds a `capture-child` integration assertion. Settle whether each is licensed by the entry's two Expected bullets or is an unlicensed extension, and whether the doc.go edits in driftcli and driftengine match the landed behavior."
  - "Test economy in spawn_test.go and baseline_test.go: settle whether the new `failures` rows and the new DriftBaseline row each add coverage the existing rows lack, naming the covering test where one duplicates it; and whether any behavior the diff adds (the lasting no-fallback marking reaching `captureAgain`, the `errors.As` ordering of the two noRecord cases) is left uncovered."
---

Seed focus for round 1. The answer key is the decision record's board entry alone; its other sections are empty, so no plan or card checks apply.
