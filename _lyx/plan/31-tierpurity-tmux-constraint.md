# Card 31 — tierpurity-tmux-constraint

**Edit:**
- `cmd/lyx/tierpurity_test.go`

**Prosa:**
- `pattern/PATTERN-test-speed.md`

**Intent:** D6's static half: the tier-purity scan bars tmuxkit spawns from a file whose build constraint lacks `tmux` and `llm`, not only from an untagged file.
`TestTierPurity_UntaggedTestsSpawnNothing` gains a second check over every tagged test file: when `tmuxkitSpawnReference` finds a reference and the file's `//go:build` line mentions neither `tmux` nor `llm`, the file fails naming the reference and the way forward, a `tmux` tag on the file or a split of the reed-starting half into a `tmux` file; the untagged check is unchanged.
A helper beside `isTierTagged` reads the constraint line's tags for the check; `TestIsTierTagged_RecognizesKnownTagsList` gains rows for it: `integration` alone is not tmux-tier, `tmux`, `llm` and `tmux || llm` are, and `integration || tmux` is.
Every tmuxkit-spawning file in the tree today carries `tmux` or `llm`, so the scan passes on landing.
`PATTERN-test-speed.md`'s "untagged tests spawn nothing" section records the rule for tmuxkit spawns.

**ImpactSummary:** One scan gains a check and one table gains rows; no production code changes.
