# Card 4 — innerrun-open-once

**Edit:**
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed/innerrun.go`

**Create:**
- `internal/battenshed/innerrun_ide_test.go`

**Intent:**
Make `Run-Shed` open the operator's IDE once per run, right after the child's bootstrap spawn succeeds, through a new injected closure, so `internal/battenshed` stays told and imports nothing new (its `seam_enforcement_test.go` must pass unchanged).

`InnerRunDeps` (`deps.go`) gains `OpenIDE func(ctx context.Context) error`, documented as: opens an editor on the task worktree with the child's session attached; called at most once per run after a spawn that returned success; its error is only warned about and never changes the row's outcome.
`NewInnerRun` resolves a nil `OpenIDE` to a no-op returning nil, the same way it resolves a nil `Sleep` and `Now`, so every existing test and non-CLI caller keeps compiling and behaving as before.

In `innerrun.go`:
- A once-marker beside the existing ones: an `ideOpenedFileSuffix = "-ide-opened"` constant and an unexported `ideOpenedFile(scratchDir, producer string) string` path helper, in the same shape as `approvalActedFile` and `doneSeenFile`.
  The marker lives in `Run-Shed`'s scratch directory, never under `_lyx`.
- An unexported `openIDEOnce(ctx)` method: when the marker exists, it does nothing.
  Otherwise it calls `p.deps.OpenIDE(ctx)`;
  on error it logs one `logger.Warn` carrying the producer, the slug and the error, and continues;
  then, whatever the outcome, it writes the marker (`MkdirAll` the scratch directory first).
  A marker write failure is logged with `logger.Warn` and not escalated, as `recordSpawnConfirmed` does, since a lost marker costs only one extra open.
- `Call`'s read-before-spawn branch: when `found` is false (a fresh child), remove any once-marker before the spawn, erroring only on a removal failure other than not-exist, the same way the done-seen marker is cleared, so a marker left by an earlier run of the same slug never suppresses this run's open.
  A running child with no confirmed spawn does not clear it.
  After `recordSpawnConfirmed`, and before the post-spawn `ReadStatus`, call `openIDEOnce`.
- `callAwaiting`: after its successful approval re-spawn's `recordSpawnConfirmed`, call `openIDEOnce`, so a run whose marker is absent (a resume on another machine, or cleared `.lyx`) opens the window there too.
- A failed spawn at either site returns before `openIDEOnce`, so it never opens.
- Extend `Call`'s disposition doc comment with the open step, its once-per-run gate and its warning-only failure mode.

Tests in the new untagged file, reusing `newInnerRunDeps` and `fakeClock` from `innerrun_test.go` with a `t.TempDir()` scratch directory and an `OpenIDE` stub counting calls:
- The first successful spawn (status absent, then running) calls `OpenIDE` once, and a call-order recorder shows it ran after `Spawn` and before the second `ReadStatus`.
- A later spawn in the same run calls it no more: a running-without-confirmation re-spawn (spawn-confirmed marker removed between calls), and an approved `awaiting` resume.
- With no once-marker present, a successful approval re-spawn in `callAwaiting` calls it exactly once.
- A once-marker pre-written into scratch does not suppress the open when `Call` finds no status file.
- An `OpenIDE` returning an error yields the same outcome, `Reason` and `BudgetExempt` as the success case, with the output captured through `logger.SetOutput` holding exactly one warning line naming the slug and the error text, and writes the marker, so the next spawn does not retry the open.
- A failed spawn does not call `OpenIDE`, at either spawn site.
- A nil `OpenIDE` behaves as before: the existing `innerrun_test.go` and `innerrun_awaiting_test.go` tests pass unmodified.

**ImpactSummary:** One new optional field on `InnerRunDeps` and a once-gated call at both spawn sites inside `innerRunProducer`; every existing constructor of `InnerRunDeps` keeps compiling and behaving identically through the nil default.
