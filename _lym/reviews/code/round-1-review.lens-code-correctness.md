```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens over the diff's changed functions and branches.

**Per-function list (case run → result)**
- `lymVerbDir`, empty root → returns `""`. Correct.
- `lymVerbDir`, task worktree `/hub/task` with marker `src/app` → `/hub/task/src/app`. Correct. It uses the same marker lookup as the cwd gate (`resolveCore` calls `readRecordedAnchor(hubPath, Base(root))`).
- `lymVerbDir`, a worktree with no own marker → falls back to the board marker, then `"."`. This is the same resolution the gate uses, so correct.
- `lymVerbDir`, unusable or unreadable marker → logs Warn and returns the root. Acceptable, since the gate would refuse that worktree too.
- `lymVerbDir`, a path that is not a direct child of the hub (`filepath.Dir` is not the hub) → falls back to the root. Not wrong, but `NamedWorktree` keys off `Dir`/`Base`, so a trailing slash or a non-hub path would resolve the wrong sibling. Callers pass clean hub-sibling paths, so no defect.
- `pushStep` records case, `CodeWorktree` set → anchored dir. Correct.
- `pushStep` records case, `CodeWorktree` empty → `""`, same text as before the diff.
- `pushStep` default (code, non-prime) and `codeDivergedWayForward` non-prime → anchored dir. Correct.
- `pushStep` and `codeDivergedWayForward`, prime → plain `git push` at the root. Unchanged and correct.
- `pull.go:442` records conflict, `f.warpPath` → anchored dir. Correct.
- `anchorStopWayForward` (slug and prime variants), `dirtyWayForward` records → `lymVerbDir(b.CodeWorktree)`. Correct. `CodeWorktree` is always set at `originreconcile.go:720-724`.
- `rewoundWayForward`, code side → git steps at the root, push in the anchored dir. Correct.
- `rewoundWayForward`, records side → unreachable. `judgeBranch` skips rewind classification when `records` is true (`originreconcile.go:604`), so `BranchRewound` is code-only.
- `stopWayForward` `BranchRewritten`, code side → anchored dir. Correct.
- `stopWayForward` `BranchRewritten`, records side → unreachable. `BranchRewritten` is set only when `!records` (`originreconcile.go:638-639`), so the focus question about the records branch resolves as follows: it never runs, and the old text there was dead.

**Findings**
- **F-A, LOW, design.** `internal/fabricengine/originreconcile.go:383-386`. The new `if b.Repo == failedPushSideCode` split has an unreachable `else`. `BranchRewritten` is produced only for code branches (`judgeBranch`, lines 604 and 638-639), so the check is always true. The records fallback still names `b.Worktree`, a records sibling that refuses every fabric verb, and would be wrong if ever reached. Evidence: `originreconcile.go:603-642`. Fix: collapse to one line, `return fmt.Sprintf("`lym fabric pull` in %s", lymVerbDir(b.CodeWorktree))`. For a code branch `CodeWorktree` equals `Worktree`, and for a records branch it would name the right directory.
- **F-B, LOW, scope** (a neighbouring case for the handler to keep or drop). `internal/fabricengine/pull.go:461-462` (`weftDirtyStop`) and `internal/fabricengine/anchorstop.go:170-171` (`upstreamAnchorStop`) still name `lym fabric commit`, `lym fabric pull` and `lym fabric anchor` in the bare `f.warpPath` or `codeWorktree`. In an anchored hub the cwd gate refuses that, so these lines are the same defect the diff fixes elsewhere. The commit message claims "every lym step now names that directory", which these two lines contradict. Fix: wrap `f.warpPath` and `codeWorktree` in `lymVerbDir(...)` at those two lines.

**Tool calls**
1. `grep BranchRewound|BranchRewritten` in `fabricengine/*.go`: to find where the states are set, which the context lacked.
2. Read `originreconcile.go:585-644` (`judgeBranch`): to confirm whether the records side can reach rewound or rewritten.
