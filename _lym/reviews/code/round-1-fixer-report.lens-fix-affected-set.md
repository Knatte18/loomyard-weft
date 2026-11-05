```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: re-ran the searches for the findings whose premise is a set of callers or sites (F1, F2, F4).

Findings left unchecked: F3, F5, F6 and F7. They are test-coverage gaps, not sets.

**F1: callers of `checkRunLocks` and `endsAsSketch`.** Every caller has the new `vocab` argument:
- `board.go:146`
- `replay.go:534`
- `replay.go:560`
- `runlock_test.go:363`
- `sketch_test.go:147`

`endsAsSketch` is called only from `runlock.go:152`. No findings.

**F4: callers of `MakeSketch`.** The only non-test caller is `wiring.go:234`, which now falls back to draft only on `ErrSketchRefused`. No findings.

**F2: sites that read the `discussed` status.**
- The guard (`startguard.go:99`) and its test row now use `LegacyStatusDiscussed`.
- `grep` finds no remaining literal or old-name reference to the status as a start-guard input.

One site the review did not name is stale:

- **[LOW]** `internal/boardcli/cli.go` and the `ly:board` skill text (`plugins/ly/skills/board/SKILL.md`, `docs/overview.md`)
  - **What:** Commit `3c153b30` reworded the `set-status` text from "null releases a draft or discussed task" to draft only.
  - **Why it is stale:** The F2 fix keeps the guard refusing a discussed task and keeps the release command naming both statuses. `releaseCommandText` and the guard's refusal text still tell the operator to release a discussed task with `set-status` null.
  - **Evidence:** Lines 116, 122 and 131 of `git show 3c153b30` show the removed wording. The guard refusal text is now "is a discussed task … release it".
  - **Suggested fix:** Restore "or discussed" in the `set-status` docs, as the review proposed for the guard's wording.

I also confirmed that the legacy-discussed layer and README section are gone:

- **[LOW]** `internal/boardengine/layer.go`
  - **What:** A discussed task that the migration cannot convert (another entry depends on it) now falls into the ordinary Ready or layer buckets. Before `3c153b30` it sat in its own Discussed layer.
  - **Why it matters:** The guard still refuses to start it, but the README shows it as ready to run. The review's F2 fix did not ask to restore the layer, so this is only a note.
  - **Evidence:** `layer.go` removals at lines 559–595 of `git show 3c153b30`.

Tool calls made:
1. `grep` for `checkRunLocks(`, `endsAsSketch(`, `CheckRunLocks(`, `legacyStatusDiscussed`, `StatusDiscussed`, `"discussed"` and `MakeSketch(` across Go files. My context had only the sites the fixer edited, not a repo-wide view.
2. `git show 3c153b30 --stat` and a grep of its removed `discussed` lines. I needed to know which sites the original commit changed besides the guard.
