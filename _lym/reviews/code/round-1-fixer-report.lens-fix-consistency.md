```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. Checking whether the change set agrees with its surroundings.

**Findings**

1. **MEDIUM** — `internal/lock/lock.go:6`. The package doc still says "Every acquire refuses a lock path inside a tracked lym directory". `refuseTrackedTree` now also refuses a lock inside a Bolt checkout, which is the F1 fix. `tracked.go:1` was updated, but this sibling statement of the same contract still describes the `_lym`-only refusal.

2. **LOW** — `internal/lymdirs/dirs.go:1-2` (file header) and `internal/lymdirs/doc.go:3-4`. Both still say the package declares "the two" worktree-level or lym directory tokens. The fix added `BoltDirName`, which is a hub-level checkout name and not one of the two. The `doc.go` list of declared names does not mention it either.

3. **LOW** — `PATTERN.md` line 13 against `pattern/PATTERN-durable-vs-ephemeral-state.md:16`. F2 asked to restore "heddle's rendered instruction files and their `.heddle` temporary directory". The overview now reads "heddle's instruction files and `.heddle`", which drops "rendered", "their" and "temporary directory". The background file still says "rendered instruction files … and the `.heddle` temporary directory". The overview is therefore not restored to its pre-`4406aef5` text, and the two disagree. The cut was forced by the 15564-byte headroom line, but the fixer-report should say so.

4. **LOW** — `PATTERN.md` line 14 against `pattern/PATTERN-tracked-file-locks.md:5,8`. The overview entry now reads "Tracked-file lock: path via `lock`. (test)". It no longer says "never beside it", nor `_lym` or Bolt, though the background still states both. The background (line 8) also does not say the Bolt refusal applies only to a `_bolt` directory holding a `.git` entry. That condition was added in `tracked.go`, and the test pins it with a `.git` file and the `worktrees/_bolt` accepted row. Neither document records it.

5. **LOW** — `internal/lymcwd/anchor.go` and `pattern/PATTERN-cwd-resolution.md:13`. The `lymcwd` import allowlist now includes `internal/lymdirs`, and the test, the pattern file and the leaf-test comment were all updated. The `lymdirs` package doc still presents the package as a stdlib-free leaf for modules "to name the two lym directory tokens", with no mention of `lymcwd`. This is the same stale text as finding 2 and is not a separate defect.

Fixed twins, no finding: the `_bolt` owner row in `enforcement_test.go`, the leaf allowlist, and the `lock` to `lymcwd` cycle avoidance.

**Tool calls**
1. `grep -rn "tracked lym\|lym directory"` over `internal/lock`, `internal/state` and `internal/lymdirs/doc.go`, plus a grep of `BoltDirName` in `internal/lymcwd/doc.go` and `PATTERN-cwd-resolution.md`. My context did not hold `lock.go`'s package doc or `lymdirs/doc.go`'s current wording. `internal/lymcwd/doc.go` does not exist.
