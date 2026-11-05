```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1–F6), looking for a concrete breakage.

1. **MEDIUM — `TrackedFileLockPath` hands back a path the new Bolt refusal rejects.**
   - Changed text: `internal/lock/tracked.go` (`TrackedFileLockPath` and the `refuseTrackedTree` Bolt branch), and the `PATTERN.md` entry "Tracked-file lock: path via `lock`".
   - `TrackedFileLockPath` mirrors a file only under a `_lym` directory. For `<hub>/_bolt/board.json` it falls through to `path + suffix`, giving `<hub>/_bolt/board.json.lock`.
   - `refuseTrackedTree` now refuses that path whenever `_bolt` holds a `.git` entry.
   - The pattern text tells callers to use `lock.TrackedFileLockPath` for Bolt files, but that call yields a refused path. Only `TrackedLockPath` works for Bolt, and the shortened entry no longer names it.
   - The review's F1 fix did not ask for a Bolt mirror in `TrackedFileLockPath`. Still, the entry and the refusal now disagree with the helper's behaviour. The entry should name `TrackedLockPath` for Bolt, or `TrackedFileLockPath` should mirror Bolt files.

2. **LOW — the Bolt refusal depends on a `.git` entry that the pattern text does not mention.**
   - Changed text: `refuseTrackedTree`, the `os.Lstat(candidate/.git)` check.
   - Case: a hub fixture or board directory named `_bolt` with no `.git` (a bare `t.TempDir()/_bolt`) is not refused. `_bolt/board.lock` then passes every acquirer.
   - `pattern/PATTERN-tracked-file-locks.md` line 8 still says "A test refuses a lock path built beside a file under `_lym` or Bolt", with no caveat.
   - The `.git` check was needed to exempt `worktrees/_bolt/lym-push.lock`. It makes the rule filesystem-dependent, and the caveat belongs in the pattern background.

3. **LOW — the F4 exemption accepts a `_lym` lock under a checkout nested below `.lym/locks`.**
   - Changed text: the `.lym` + `locks` early return in `refuseTrackedTree`.
   - Case: `/x/.lym/locks/co/_lym/record.json.lock`, a checkout living under some `.lym/locks` ancestor, returns nil at the first `.lym`/`locks` pair. A `_lym` lock beside a tracked file in that checkout is therefore not refused.
   - The review asked for exactly this exemption, so the gap is the proposal's own. It is narrow, and no test pins it.

4. **LOW — the F2 restoration is not the exact text the review asked for.**
   - Changed text: `PATTERN.md` line 13 now reads "heddle's instruction files and `.heddle`". The review asked for "rendered instruction files and their `.heddle` temporary directory".
   - The fixer dropped "rendered", "their" and "temporary directory" to fit the headroom line. `pattern/PATTERN-durable-vs-ephemeral-state.md` still says "rendered instruction files … and the `.heddle` temporary directory", so the two now differ slightly.
   - The review's budget was arithmetically unreachable, and the background link could not be dropped (it triggers `orphan-background`). The fixer should record this as "fixed differently" in its report.

No further tool calls were made; my context already held every file the lens needed.
