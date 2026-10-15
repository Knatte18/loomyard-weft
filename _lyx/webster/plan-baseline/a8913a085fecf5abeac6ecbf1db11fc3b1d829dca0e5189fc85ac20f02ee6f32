# Card 1 — settings-golden

**Create:**
- `plan:internal/shuttleengine/claudeengine#TestBuildSettings_MatchesGolden` -> `func TestBuildSettings_MatchesGolden(t *testing.T)`
- `internal/shuttleengine/claudeengine/testdata/settings-golden.json`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Pin today's Claude settings document and deny notice byte for byte before any later card refactors them, so the decision record's criterion that every Claude session's hooks and notice stay unchanged (D6) has a test that fails on any drift.

`testdata/settings-golden.json` holds one entry per combination of the six deny inputs: interactive, fork subagents, allow-Agent-tool, the Agent deny, the AskUserQuestion deny and the python deny, every one of them false and true, so 64 entries.
Each entry is keyed by a stable label naming the six values, such as `interactive=false fork=true allowAgent=false denyAgent=true denyAsk=true denyPython=false`, and holds two strings: `settings`, the exact bytes `buildSettings` returns for the events path `/c/run/events.jsonl` and an empty context-after-compaction command, and `notice`, the exact string `buildDenyNotice` returns for the same inputs.

The file is generated once in this card from the code as it stands, before any change to `settings.go`: the implementer writes a throwaway generator test that marshals the 64 entries with `encoding/json`, sorted keys and two-space indent, runs it once with `lyx gate test ./internal/shuttleengine/claudeengine`, and deletes the generator in the same card, so the committed file is today's output and no update mode ships.

`TestBuildSettings_MatchesGolden` reads the file, renders the same 64 combinations through `buildSettings` and `buildDenyNotice`, and fails naming the label when the settings bytes or the notice differ from the entry, and when the file holds a label the loop does not render or the loop renders one the file lacks.
Later cards that change the signatures of `buildSettings` and `buildDenyNotice` or rename a deny input edit this test's inputs, never the golden file.

`claudeengine` already sits at its row in `cmd/lyx/testdata/test-budget.yaml`, so this card raises the row from 112 to 113.

**ImpactSummary:** One new test and one testdata file in `claudeengine`, and the package's budget row raised by one; no production code changes.
