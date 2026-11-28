```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. Trace each change in the diff to the board entry's goal points.

**Decisions.** The answer key is the decision record alone. It has no `Added after Discussion` entries, and the board entry under `## Goal` holds two goal points:
- **G1:** stop the WARN that `lym board sync` from the prime logs, one per long-removed task worktree, from `TargetOf`'s prime read (`resolve main worktree … no .git above <hub>/<slug>`).
- **G2:** "Expected: entries for removed pairs are dropped at removal (or skipped silently when the directory is gone)."

**Changes and what each traces to:**
- **C1:** `worktreeGone` moves to `targetreads.go`, takes a path and decides on `<path>/.git` (`targetreads.go:81-95`). `Run`'s `WorktreeGone` seam now calls it (`run.go:98`), and the old directory-only helper is removed.
  - Traces to G1.
  - The reported message, "no .git above <hub>/<slug>", comes from a path that still exists without `.git`, a leftover directory. A directory-only check would not catch it.
  - The `Run` seam switching to the shared helper is forced by the single definition.
- **C2:** `starter.pairGone` is read first for a pair, which then starts none, reads nothing else and writes no ledger entry (`start.go:45-46`, `97`, `119-124`). Traces to G2, the "skipped silently" branch.
- **C3:** `TargetOf` returns a pair target with nil error for a gone worktree, before `IsPrimeWorktree` (`target.go:72-80`). Traces to G1, because the warned error came from `IsPrimeWorktree` through `TargetOf` in `restartPusher` (`commitstatus.go:202-210`, reached from `dobbyreads.go:323-324`).
- **C4:** `TargetRefused` answers false for a gone pair (`targetreads.go:99-106`). Traces to G1 and G2: the refusal read in the start and in `restartHubPushers`/`settleIfRefused` (`dobbydeps.go:265-267`) would otherwise run git on the removed path and fail the same way.
- **C5:** `PruneLedger` also drops the entry of a leftover directory without `.git` (`targetreads.go:136-162`). Traces to G2, "entries for removed pairs are dropped".
- **C6:** the doc comments (`doc.go:15-20`, `start.go:65-66`, `target.go:72-74`, `targetreads.go:99`, `136-137`) and the tests (`seams_test.go`, `start_test.go` rows, the `TestPruneLedger` leftover row). All are forced by C1–C5.

**Decisions and the changes that build them:**
- **G1:** built by C1, C3 and C4, with C2 as the silent-skip backstop.
- **G2:**
  - The "skipped silently when the directory is gone" branch is built by C2 and C3.
  - The "entries … dropped" part is built by C5, for push-ledger entries.
  - The "dropped at removal" branch is not built. The goal offers it as an alternative ("or"), so leaving it out is no gap.
  - The batten run records that `DobbyRuns` iterates are not pruned. The silent skip satisfies the alternative the goal allows.

**Findings:** none. Every hunk traces to G1 or G2 or is forced by a change that does. No goal point is left unbuilt beyond the alternative the entry itself makes optional. The range carries no `Pattern-Departure:` trailer, and no change needs one.

Tool calls: none.
