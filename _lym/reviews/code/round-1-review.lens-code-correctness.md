```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, covering each function or branch the diff changes, the case I ran, and whether it was correct.

**Per-function list**

- `renderEveryWorktree`, prime and beam entries: the git-listed path's base name goes through `lymcwd.NamedWorktree(hub, name)` and renders into the hub-level path. Correct for a worktree directly in the hub, as the prime, beams and finished pairs are.
- `renderEveryWorktree`, prunable entry: skipped. Correct.
- `renderEveryWorktree`, entry for a pair still being made, such as a path under `.fabric-wt-staging-*/<leaf>`: **not correct**, see finding 1.
- `renderEveryWorktree`, `List` failure: returns one warning with `rendered` and `failed` nil. It does not crash, but a caller reading `failed` sees nothing wrong. See finding 2.
- `renderEveryWorktree`, `Unwritten` branch: the cause is built from `report.Warnings`. Every `Unwritten` append in `Render` (`internal/heddleengine/render.go:143-153`) comes with a warning, so the cause is never empty. Correct on the cause. The way forward is only whatever the warning text carries, see finding 3.
- `renderEveryWorktree`, `err != nil` branch (`NamedWorktree`, `EventFor` or `Render` failing): the entry goes to `failed` with a way forward. Correct, with a cosmetic duplicate, see finding 3.
- `finishWrite`, `render=false`: no `rendered` or `failed` field, and the sync still starts. Correct.
- `finishWrite`, `render=true`: `failed` is normalised to `[]` and `rendered` goes through `nonNil`. `render_warnings` appears only when there is something to report. Correct.
- `finishWrite`, refused or failed save: `fail` is called and neither render nor sync runs. Correct, and the same as the old `write`.
- `finishWrite`, ordering: `saveLocked` releases the Bolt lock through its `defer` before the fan-out, and the sync starts after it. Correct, but a still-running detached sync from an earlier write can hold the lock, see finding 4.
- `syncAfterSave`: a sync-start error is logged and returned as a warning. Same behaviour as the old `write` plus `syncWarning`. Correct.
- `changeStore` and `ChangeStore`: save, then sync, no render, warning returned. Dropping the `syncWarning` field is correct. Without a shared field there is no stale warning left over.
- `setOverride` and `removeCmd` with `--layer lym`: `render` flows to `finishWrite` on every path. Correct. The override store is saved before the fan-out, so `Render` sees it.
- `renderCmd` via `renderInto`: the behaviour is unchanged, except that the seed-exclude failure message changed. See finding 3.

**Findings**

1. MEDIUM, design. `internal/heddlecli/verbs.go:458-466` (`renderEveryWorktree`).
   - It skips only `Prunable` entries and takes `filepath.Base(entry.Path)` as a hub worktree name, with no check that the path lies in the hub.
   - Other hub code that lists worktrees skips `fabricengine.InWorktreeStaging(entry.Path)` first: `internal/orchcli/live.go:32`, with the reason given in the comment at lines 28-31.
   - Case run: a pair mid-creation is listed under `<hub>/.fabric-wt-staging-X/<leaf>`. The name becomes `<leaf>`, and `NamedWorktree` builds `hub/<leaf>`, which does not exist.
   - `EventFor` then runs `SeedWarpExclude` on that missing worktree and fails. The pair is reported in `failed` under a name it will not keep.
   - The fan-out can also write exclude and heddle scratch state at wrong hub-level paths. I did not verify that last point.
   - Fix: skip staging entries, and skip any entry whose path is not directly in `HubPath`.

2. LOW, design. `internal/heddlecli/verbs.go:454-457`.
   - When the worktree list fails, nothing is rendered and `failed` stays `[]`. The only signal is an entry in `render_warnings`.
   - Output that is clean in `failed` while no worktree was rendered misleads a caller that checks `failed`, which the entry's "report the failed worktrees" wording asks callers to do.

3. LOW, consistency. `internal/heddlecli/verbs.go:470-472`, `internal/heddlecli/doc.go` (hunk), `internal/heddlecli/verbs.go:365-371`.
   - The `Unwritten` case carries no `lym heddle render <worktree>` way forward of its own, although `doc.go` says each failed worktree comes with it. Only the warnings' text, which some warnings carry, supplies one.
   - The `err` case appends the way forward unconditionally. A `Render` error that already names `lym heddle render` gets it twice. The `render` verb guards against this with its `strings.Contains` check, and the fan-out does not.

4. LOW, design. `internal/heddlecli/verbs.go:412-433`, `internal/heddleengine/render.go:185-190`.
   - Every worktree's `Render` takes the Bolt lock to read the store, and each waits up to the default lock wait.
   - A detached sync from a previous `set` in a run of edits can still hold the lock. Then each worktree can wait the full time in turn, and each is reported failed, so a repeated `set` run takes long and fails the fan-out.
   - The change is saved either way, so nothing is lost. I did not verify how long a sync holds the lock.

No state is left inconsistent on any failure path I ran. The save always happens before the fan-out, and a failed fan-out leaves the sync still starting.

**Tool calls:** none. My inherited context already held every definition I needed.
