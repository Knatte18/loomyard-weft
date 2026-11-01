```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals of F1, F2 and F3, each of which the fixer carried out as proposed.

**Findings**

1. **LOW** — F2, `Decode` in `internal/heddleengine/store.go`. The way forward "fix the file in a new commit on the local branch" is wrong for two of the three blobs.
   - `Replayer.Apply` (`replayer.go:63-73`) also decodes the merge base and the parent. These come from history and from origin. A new local commit cannot repair either of them. At best it repairs the replayed commit's own blob.
   - Concrete case: origin's `heddle.json` is malformed, so `parent` fails to decode. The operator is told to commit a fix locally, but the replay will read the same bad parent blob again.
   - The reviewer's own text said "which side failed". The new message names no side itself, and the replayer's wrapper prefixes only say "merge base", "parent" and "commit". The advice is one-size-fits-all and misleads for base and parent.

2. **LOW** — F3, `docs/overview.md:172`. Replacing the literal `_board` with "the Bolt checkout's directory" in the `HubReservedNames()` token list loses precision.
   - The reserved token is a literal name. `lymcwd.BoltDirName = "_board"` (`internal/lymcwd/anchor.go:25`) is still `_board`, so the old text was correct there.
   - A reader can no longer see which token is reserved, next to `_portals` and `_launchers`, which stay literal.
   - The reviewer asked to reword only where the text describes the checkout rather than the board's own files. A reserved-token list is the case that needs the literal.

3. **LOW** — F3, left stale in `docs/overview.md:173, 375, 376`. Line 172 now says `<BoltDir>/_lym/config/fabric.yaml`. The same section still uses `<BoardDir>/_lym/config/...` for the repo default, the board config and the machine override. Per the shown `ConfigDirs` doc, those configs also come from the Bolt checkout. The fix leaves the page using two names for one directory. `BoardDir` and `BoltDir` currently resolve to the same path, so this is a naming inconsistency and not a wrong path.

**Checked, no finding**
- F1, empty `--targets`. The non-nil empty slice passes `Set`'s `p.Targets != nil` branch, so `ValidateEntry` rejects it with a message naming `--targets`. A new entry given an empty list is refused the same way, and `--targets ","` also reaches `ValidateEntry` and is refused.
- F1, rename. No other source string or test depended on `--target`. The tightened `"--targets"` row would fail on the old text.

**Tool calls:** none. The context already held every file and grep output used above.
