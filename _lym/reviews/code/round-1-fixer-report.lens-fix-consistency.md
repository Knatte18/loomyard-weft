```yaml
lens: fix-consistency
outcome: reported
```

No findings.

The changed text agrees with its surroundings:
- The `docs/overview.md:362` entry matches `pattern/PATTERN-leaf-packages.md:97-98`, which admits `internal/standalonestate`, and the `TestLeafInvariant_AllowlistOnly` allowlist.
- The `internal/standalonestate` entry still says "stdlib-only leaf", which stays true, so the added "also resolving the per-user config directory" clause leaves nothing stale.
- `pattern/PATTERN-owner-machine.md:74` speaks only of the sole-source hostname scan, and that is unchanged.
- The only other hit for "machinename" as a stdlib-only leaf is a historical benchmark report, which records a past run and is not meant to stay current.
- In `TestFilePath`, the rows cover both platforms' variables, so `relativeVariable` matches the variable named in the error on each. The "absolute path" text matches `FilePath`'s wrap at `internal/machinename/machinename.go:45`.
- The new `TestUserConfigDir` row has the same shape as its siblings.

Tool calls:
1. grep of `*.md` and `doc.go` for stale "stdlib-only" or hostname-only claims about `machinename`: my context held only `docs/overview.md`.
2. Read of `pattern/PATTERN-leaf-packages.md` lines 90-110: my context didn't hold the pattern entry the overview must agree with.
