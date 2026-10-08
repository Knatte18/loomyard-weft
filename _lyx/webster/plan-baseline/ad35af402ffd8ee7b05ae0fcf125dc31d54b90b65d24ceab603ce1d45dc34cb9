# Card 1 — deny-table

**Edit:**
- `internal/shuttleengine/claudeengine/settings.go`
- `internal/shuttleengine/claudeengine/settings_test.go`

**Intent:**
Declare lyx's standing denies in one table in `settings.go` (D10), and make `buildSettings` and `buildDenyNotice` both iterate it, replacing `denyInstalls`.
Each row names its tool matcher (`Agent`, `Bash`, `AskUserQuestion`), its hook command (the match and the deny JSON with its steer), its system-prompt notice (empty for a row with none, as the webster fork guard has today) and an install predicate over the inputs that decide it today: `interactive`, `cfg`, `forkSubagents`, `allowAgentTool`.
The existing denies move in unchanged in behavior: the Agent deny and its fork-mode variant, the AskUserQuestion deny (autonomous runs only), and the webster fork guard.
The interactive AskUserQuestion record hook is a recording hook, not a deny, and stays outside the table, written by `buildSettings` as today.
The `init` forbidden-character check iterates the table's steers and notices instead of its hand-kept lists, so a new row is checked without a second edit.
Every hook command and notice sentence, and the order of the `PreToolUse` entries, stays byte-identical to today's.

Test: `settings_test.go`'s existing rows pin the documents and notices byte for byte, and no new test is added.
`TestBuildDenyNotice_MatchesInstalledDenies` already asserts, over every deny combination, that the notice announces exactly the denies `buildSettings` installs; its hand-kept `wantAgent`/`wantAsk` switch is generalized to read each installed `PreToolUse` entry back to its table row and expect that row's notice sentence, so the drift guard is read off the table.
`TestBuildSettings_NoForbiddenCharsInSteerText` keeps covering the forbidden-character check, now over the table's steers and notices.

**ImpactSummary:** Pure restructuring of claudeengine's settings composition; settings.json and the deny notice stay byte-identical for every input.
