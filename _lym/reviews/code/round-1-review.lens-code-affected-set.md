```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change in the diff I list what reads, matches or documents it outside the touched lines.

**Set per change**

1. **Run name is now `r` plus four of `[0-9a-z]`.**
   - Readers of the name:
     - `internal/lymtemp/sweep.go:54` matches `runPrefix` only.
     - `internal/lymtemp/root.go:54` checks `runPrefix` only, for the inherited-run-directory case.
     - `internal/lymtemp/root_test.go:153-154` builds fixtures as `runPrefix+"123456"`, which is prefix-only and still valid.
     - `run_test.go:32` is the regexp, updated.
     - `run_test.go:74-76` uses `rtaken`/`rfresh` stubs through the `root.runName` seam, with no assumption about the format.
   - A repo-wide search for six-hex assumptions (`r<hex>`, `six hex`, `[0-9a-f]{6}`) found no code consumer left on the old shape. Covered.
   - Stale spots:
     - `internal/testkit/tmuxkit/tmuxkit_test.go:34` still uses `/tmp/lym-4294967295/r0a1b2c`, the old six-hex shape, as a fixture. Not covered; this is finding 1 below.
     - `docs/benchmarks/test-suite-timing.md:736` and `:767` still say `r49ee2f` and `r<hex>`. They sit in the earlier `fabricengine-shm-bench` measurement report, so they are exempt as a historical record. Covered.
2. **Docs of the format.**
   - `internal/lymtemp/doc.go` and `pattern/PATTERN-lym-temp-root.md:7` are both updated, and nothing else describes the name. The `PATTERN.md` index line for `PATTERN-lym-temp-root` names no format, so it needs no edit. Covered.
   - The directive's declared PATTERN changes list is empty, so no landing check is owed. Covered.
3. **`checkSocketPath` and the new `TestCheckSocketPath` row.**
   - The only caller is `tmuxkit.Main`, and `checkSocketPath` itself is unchanged. The row builds its directory from `lymtemp.Dir()`'s basename.
   - Arithmetic from the code: `/dev/shm` is 8 bytes, `/lym-1856216086` is 15, `/rxxxx` is 6, `/tmux-1856216086` is 16, and `/` plus the 61-byte key is 62. The total is 107, and the `>` comparison at `tmuxkit.go:451` admits it. Covered.
4. **Commit messages.** Neither commit carries a `Pattern-Departure:` trailer, and none is needed because no listed entry is forbidden. The 109-to-107 claim matches the arithmetic. Covered.

**Findings**

1. NIT, `consistency`. `internal/testkit/tmuxkit/tmuxkit_test.go:34`: the "run directory of the widest uid" row keeps the old name `r0a1b2c`, six hex characters. The row's length is no longer what a real run directory has, so it is the one fixture the diff leaves on the old format.
2. NIT, `consistency`. `internal/lymtemp/run.go:24-27`:
   - Two comment problems:
     - The `runNameDigits` comment pins a perishable tally ("over a million names") that `len(runNameAlphabet)` and `runNameDigits` already derive.
     - It also names `/dev/shm`, the ten-digit uid and the 107-byte limit, which belong to `tmuxkit`, not to `lymtemp`.
   - The name `runNameDigits` counts letters as well as digits, so `runNameLength` would fit better.

**Tool calls:** none. My inherited context already held every definition and the search results.
