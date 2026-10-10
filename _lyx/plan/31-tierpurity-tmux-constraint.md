# Card 31 — tierpurity-tmux-constraint

**Edit:**
- `cmd/lyx/tierpurity_test.go`

**Prosa:**
- `pattern/PATTERN-test-speed.md`

**Intent:** D6's static half: the tier-purity scan bars tmuxkit spawns from a file whose build constraint lacks `tmux` and `llm`, not only from an untagged file.
`TestTierPurity_UntaggedTestsSpawnNothing` gains a second check over every tagged test file: when `tmuxkitSpawnReference` finds a reference and the file is not tmux-tier, the file fails naming the reference and the way forward, a `tmux` tag on the file or a split of the reed-starting half into a `tmux` file; the untagged check is unchanged.
A file is tmux-tier when it compiles in neither the untagged nor the `integration` build on any platform: a helper beside `isTierTagged` parses its `//go:build` line with `go/build/constraint`, holds `tmux` and `llm` false, and evaluates the expression under every assignment of its other tags, `integration` and platform tags such as `windows` included; the file is tmux-tier when every evaluation is false.
The host's `runtime.GOOS` plays no part, so a file the scan accepts on Linux cannot start a server in a Windows tier-2 run, and the tag count of a build line keeps the enumeration small.
Mentioning `tmux` is therefore not enough: an `integration || tmux` file compiles in the `integration` build, so a tmuxkit spawn in it would start a server in tier 2, which is the hazard card 6 names for loomcli's `smoke_helpers_test.go`.
`TestIsTierTagged_RecognizesKnownTagsList` gains rows for the helper: `integration` alone, `integration || tmux` and `integration && windows` are not tmux-tier, and `tmux`, `llm`, `tmux || llm` and `tmux && !windows` are.
Every tmuxkit-spawning file in the tree today is constrained to `tmux`, `llm`, `tmux || llm` or `tmux && !windows`, and card 6 adds registrations only to files of those shapes, so the scan passes on landing.
`PATTERN-test-speed.md`'s "untagged tests spawn nothing" section records the rule for tmuxkit spawns.

**ImpactSummary:** One scan gains a check and one table gains rows; no production code changes.
