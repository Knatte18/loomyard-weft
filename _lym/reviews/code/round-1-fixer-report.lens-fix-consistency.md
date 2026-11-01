```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1–F3 change set for consistency with its surroundings.

**Findings**

1. **LOW** — `docs/overview.md:375,376`, a twin of the line-172 edit that was left as it was.
   - Line 172 now says `<BoltDir>/_lym/config/fabric.yaml`.
   - Lines 375 and 376 still give `<BoardDir>/_lym/config/<module>.yaml` and `<BoardDir>/.lym/config/<module>.machine.yaml` as the repo default and machine override of config modules.
   - `BoardDir` is documented in `junctionnames.go` as the board's own data (board.json and its overview and design files). `_lym/config` lives in the Bolt checkout, so the finding's own premise applies to these two lines.
   - Line 173 (`board.yaml` at `<BoardDir>/_lym/config/board.yaml`) is arguably a board file and defensible.
   - The review named only lines 119, 126, 143, 182 and 370 for rewording, so this is a stale sibling, not a missed instruction.

2. **LOW** — `docs/overview.md:126`, left unedited though the review's fix named it.
   - The line still reads `` `_board`, `_launchers` and the retired `_portals` are hub geometry``.
   - Line 172 was reworded to "the Bolt checkout's directory" and line 126 kept the literal, so the two disagree in style.
   - Line 126 names the reserved slug token, which is the literal directory name, so keeping `_board` there is defensible.
   - The fixer should say in the report that line 126 was kept on purpose, since the review listed it.

3. **LOW** — `docs/overview.md:172`, the new phrase "the Bolt checkout's directory".
   - It replaces the concrete reserved token `_board` in a list of `HubReservedNames()` tokens.
   - The list is now less concrete than its `_portals` and `_launchers` siblings. Consider "`_board` (the Bolt checkout)".

I found no other mismatch.
- The `--targets` strings in `store.go` match the flag declared in `verbs.go`.
- `decodeFrom` has no remaining references.
- The `store_test.go` row `NewWithoutTargets` was tightened to `--targets`.
- The `ValidateEntry` message "give at least one of %s in --targets" is not asserted elsewhere.
- Other heddle help text still says `--targets`.

**Calls made**
1. `git diff HEAD~2 -- docs/overview.md` plus a grep for `decodeFrom` and `--target`: my context lacked the committed doc diff and any grep of those strings across `internal/heddle*`, `docs` and `contracts`.
2. A grep for `splitTargets` and `unchanged` in `internal/heddlecli`, and for `<BoardDir>` in `docs`: my context lacked the other callers of `splitTargets` and the remaining `BoardDir` mentions in the docs.
