# Card 4 — Deny the hold takeover to agents

**Create:**
- `internal/shellguard#adoptTakeHoldCommandPattern`
- `internal/shellguard#steerAdoptTakeHoldDeny`
- `internal/shellguard#noticeAdoptTakeHoldDeny`

**Edit:**
- `internal/shellguard#rowsAfterRawGo`
- `internal/shellguard#TestRows_PatternsDenyAndAllow`
- `internal/shuttleengine/claudeengine#TestBuildSettings_DenyToggleMatrix`
- `internal/shuttleengine/claudeengine#TestPrepare_WritesArtifactsAndReturnsConsistentLaunch`
- `internal/shuttleengine/claudeengine#TestBuildSettings_NoAgentHookAndForkGuard`
- `internal/shuttleengine/claudeengine/testdata/settings-golden.json`

**Uses:**
- `internal/shellguard/doc.go`
- `internal/shellguard#Row`
- `internal/shellguard#TestCheckTexts`
- `internal/shuttleengine/claudeengine#TestBuildSettings_MatchesGolden`

**Intent:**
A row of `internal/shellguard`'s table (`PATTERN-shell-guard-table`) denies `lym adopt --take-hold` to every agent.
It goes in `rowsAfterRawGo` as `Name: "adopt-take-hold"`, `Switch: SwitchNone`, `Patterns: []string{adoptTakeHoldCommandPattern}`, `Forms: [][]string{{"lym", "adopt", "--take-hold"}}`.
The pattern matches `lym adopt` in command position followed anywhere on the line by the `--take-hold` flag word, `--take-hold=...` included; its comment states what it lets through: a written script, a quoted command substitution, the flag spelled through a variable.
The steer and notice say only that an adopt's takeover of another machine's hold is the operator's call; the steer's way forward is to ask the operator.
Neither spells the flag, and both pass `checkTexts`.
The pattern and the `Forms` entry are the only places the row names the flag; `cmd/lym`'s no-denied-recovery scan reads the form with no edit.
Bound: a guardrail over static text (`internal/shellguard/doc.go`); a bypass still meets the hold rules, no skipped check and no run before a green check.

`TestRows_PatternsDenyAndAllow` (tier 1) gains rows: `lym adopt --take-hold b` denied, `lym adopt b --take-hold` denied, `lym adopt --take-hold=yes b` denied, `cd x && lym adopt --take-hold root` denied, `lym adopt b` allowed, `lym adopt --here` allowed, `echo lym adopt` allowed.
The claude engine renders every row, so its assertions move:
`TestBuildSettings_DenyToggleMatrix` (tier 0) expects six standing Bash denies, the new one asserted installed under every input;
`TestPrepare_WritesArtifactsAndReturnsConsistentLaunch` (tier 0) moves its `wantBash` counts from 6 and 5 to 7 and 6;
`TestBuildSettings_NoAgentHookAndForkGuard` (tier 0) gives the new row its own bucket, matched by its steer, with a want-exactly-one assertion, so the default bucket keeps its want-none and want-the-fork-guard checks;
`settings-golden.json` gains the rendered row, which `TestBuildSettings_MatchesGolden` compares.

**ImpactSummary:** one table row; the claude engine's three settings tests and its golden file move with it, and the codex engine derives its rows from the table.

**Commit:** 4: Deny `lym adopt --take-hold` to agents through a shell-guard row
