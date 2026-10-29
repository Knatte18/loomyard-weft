```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds, the test row that pins it:

**`cmd/lym/interactivespawn_test.go`**
- Call shape, claude literal with `-p`: pinned by fixture row "Command with a claude literal and -p".
- Call shape, claude path with `--print`: pinned by "CommandContext with a claude path and --print".
- Call shape, `--print=…` with an identifier naming claude: pinned by "claude variable with --print=".
- Call shape, selector naming claude: pinned by "claude config selector".
- Call shape, call naming claude: pinned by "claude binary call".
- Token rule, unquoted line: pinned by "pane line string".
- Token rule, quote trimming and a Windows base name: pinned by "quoted claude.exe in a pane line".
- Token rule, print flag before claude: pinned by "print flag before claude" (0 failures).
- Claude-engine directory rule: pinned by "print flag fragment in the claude engine" (1 failure) and "print flag fragment outside the claude engine" (0).
- Clean tree and tmux's `-p`: pinned by "interactive launch" and "tmux -p".
- Comment never read: pinned by "mention in a comment".
- Test files not scanned: pinned by "test file is not scanned".
- The scan passing on the current tree and the file-count floor: pinned by `TestInteractiveSpawn_NoHeadlessClaude`.
- Allowlist stale-entry reporting: wired in `TestInteractiveSpawn_NoHeadlessClaude`, but with an empty list. Finding 1.
- Allowlist admission, `allow.Allowed(rel)` at line 114: none. Finding 1.

**`cmd/lym/fslinkonly_test.go`**
- `os.Symlink` and `os.Link` found: pinned by "Symlink" and "Link".
- Aliased `os` import: pinned by "aliased os import".
- Call inside `internal/fslink` passes: pinned by "call inside fslink".
- Comment passes: pinned by "mention in a comment".
- `Readlink` is not a link call: pinned by "Readlink".
- Test file not scanned: pinned by "test file is not scanned".
- `<file>:<call>` allowlist admitting `tools/deploy/main.go:os.Symlink`, and its stale check: pinned by `TestFslinkOnly_NoRawLinkCalls`, because that entry matches a real site and the test would fail if it were missing or stale.
- Dot or blank `os` import (`osImportName` returns `""`, so the file is skipped): none. Finding 2.

**`cmd/lym/tierpurity_test.go`**
- The new `allowedSpawners` entry: pinned by the existing tier-purity scan, which would flag the file if the entry were missing.

**`PATTERN.md` entries and `pattern/` background files**
- Not mine: `pattern.Check` and the verify gate decide them. They are static prose.

Findings:

1. **LOW, class `design`.** `cmd/lym/interactivespawn_test.go:114`.
   - The allowlist-admission branch, `if allow.Allowed(rel) { return }`, has no test row. The production test passes an empty `allowedHeadlessSpawn`, and every fixture row passes `NewAllowlist(nil)`.
   - Deleting that line would break nothing, so D2's "an allowlist keyed `<file>` admits a site" is untested.
   - The fix is one row with a non-nil allowlist, for example `NewAllowlist([]scankit.Entry{{Key: "p/a.go", Why: "…"}})` over a violating file with 0 failures expected.
   - The `fslink` allowlist does not have this gap, because its real entry exercises the same path.

2. **LOW, class `design`.** `cmd/lym/fslinkonly_test.go:85-93`.
   - `osImportName` (`cmd/lym/baretemp_test.go:32`) returns `""` for a dot or blank `os` import, and `linkCallFailures` then skips the whole file. A file with `import . "os"` and a bare `Symlink(...)` passes the scan.
   - No fixture row records whether this is intended. The focus asks how every way of binding `os` is handled, and D4 says "any name the file binds `os` to".
   - Either add a row that pins the bound (dot import, 0 failures) or have the scan handle it. A blank import binds no name, so it needs no row.

Tool calls: none. My context already held all the files.
