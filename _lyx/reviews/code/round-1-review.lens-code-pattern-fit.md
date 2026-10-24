```yaml
lens: code-pattern-fit
outcome: reported
```

Task: lens code-pattern-fit. List of rules checked, held or broken.

**Rules checked**
- `PATTERN-tier-tags`: held. The diff drops `|| integration` from the guard files, and the one-tag scan now refuses `integration`, `tmux` and `llm` in any file.
- `PATTERN-shell-guard-table`: held. The raw go row in `internal/shellguard/rows.go` changed in place and the `claudeengine` settings golden followed it.
- `PATTERN-hub-wide-config`: held. The hub-wide `gate.yaml` on the board already carries single tier tags (`_board/_lyx/config/gate.yaml` lines 7, 15, 24, 33). `internal/gatecli/doc.go` follows it.
- `PATTERN-test-economy`: held. Fixture rows were respelled or dropped in place, and I saw no duplicate tests.
- `PATTERN-comment-line-breaks`, comment conventions: held in the diffs I read. Each doc comment is one sentence per line and none restates history.
- Commit message: held. The diff has one commit, with no `Pattern-Departure:` trailer, and none is needed.
- Declared PATTERN changes: the directive declares none, so nothing to check.
- `PATTERN-test-speed`: broken. See finding 1.
- `PATTERN-test-isolation`: broken. See finding 2.
- CLAUDE.md "Docs land in the same commit": broken for the two `pattern/` files above. The diff touches no file under `pattern/` or `PATTERN.md`.

**Findings**

1. MEDIUM, class `consistency`, `pattern/PATTERN-test-speed.md:19`.
   - The line still says a `//lyx:guard` file takes `tierN || integration` until the deployed `lyx` replaces the guard scan, and that the hub's tier commands carry both spellings "for that transition".
   - This diff ends that transition: the guard files, the tier commands and the one-tag scan all drop the old term.
   - The line now describes a state the code refuses, and `cmd/lyx/tiertags_test.go` now fails a `tier0 || integration` guard file.
   - It needs a rewrite to "a guard file takes its one `tierN` tag" (evidence: `grep` output above, line 19).

2. MEDIUM, class `consistency`, `pattern/PATTERN-test-isolation.md` lines 14, 27, 28, 47 and 48.
   - Line 14 says the `TestMain` rule holds "under every tag set (untagged, each of `integration`, `tmux` and `llm` alone, and the three together)".
   - The diff's `cmd/lyx/tmuxisolation_test.go` changes `isolationTags` and `isolationTagSets` to tier tags only, so line 14 no longer matches what the guard enforces.
   - Lines 27 and 28 name `tmux`- and `llm`-tier tests and "an `integration` build".
   - Lines 47 and 48 say "built without the `tmux` and `llm` tags" and tell the author to "tag the file `tmux`".
   - That last instruction points at a spelling the one-tag scan now rejects.
   - The `tmuxkit.Main` socket check itself is outside this diff, so I flag only the prose that cites the old tags (evidence: `grep` output above).

3. LOW, class `consistency`, `contracts/stencils/govalues_test.go:102`.
   - The `banned` list still holds `"-tags tmux"` and `"-tags llm"` next to the other banned strings.
   - The decision record's "comments outside the touched files keep their old spellings" covers prose, but this is a test fixture in a file the diff does touch (one line, `|| integration` removed).
   - After this change the stencils' banned-command check no longer covers the tier-tag spellings (`-tags tier2`, `-tags tier3`) that the raw go row now refuses.
   - Whether to add them is a judgment call.
   - I read only the grep hit, not the file around it.

**Tool calls**
1. `grep` over `pattern/PATTERN-test-speed.md`, `PATTERN-test-tiers.md` and `PATTERN-test-isolation.md` plus a `git diff --stat` of `PATTERN.md` and `pattern/`. My context had no `pattern/` background text, and the stat printed nothing, so the diff touches neither.
