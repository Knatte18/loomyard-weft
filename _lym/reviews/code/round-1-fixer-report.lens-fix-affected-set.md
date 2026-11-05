```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Findings I checked:
- **F2** (verb comments name only the Claude engine): I searched `internal/creelcli` and `internal/creelengine` for comments tying `hook-wait`, `hook-read`, the verb or "spell" to the Claude engine. Nothing is left beyond the two rewritten comments.
  - The only other hits are `internal/creelcli/cli_test.go:157` and `:163`. They say "the claude engine's hook output", which describes what the test fixture prints, so they are accurate and don't count as stale.
- **F4** (leftover indirections in `claudeengine`): I searched `internal/shuttleengine/claudeengine` for `strandFlagName`, `creelTrustSentence`, other `= shuttleengine.X` constant aliases, and other one-line `func x() string` wrappers.
  - `strandFlagName` and `creelTrustSentence` no longer appear anywhere.
  - No similar alias or wrapper remains. The `CreelFlagDir` call sites already use `shuttleengine.CreelFlagDir`, in `claudeengine.go:272` and `settings_test.go:684`.

Findings I left unchecked:
- **F3** has a test-coverage premise, not a set.
- **F1** is `decision`-class and went to the parent.

Tool calls:
- Search 1 (grep of `creelcli` and `creelengine` for engine-naming verb comments): reran F2's own search, which my context lacked.
- Search 2 (grep of `claudeengine` for leftover shims and aliases): reran F4's own search, which my context lacked.
