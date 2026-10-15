# Relevant code

- `tools/tokencount/main.go` — package comment documents every report rule, the `-projects` flag and the projects-dir resolution
- `tools/tokencount/count.go` `line`, `Usage`, `CountRun`, `countFile`, `lastLinePerMessage`, `endsTurn`, `startsTurn`, `readMerriamStart`, `projectDir`, `roleOf`, `latestTitle`, `Report.WriteMarkdown` — the Claude transcript parser to delete and the tallies that stay
- `tools/tokencount/cost.go` `prices`, `messageCost`, `Cost` — the price table that moves and the cost tally that stays
- `tools/tokencount/orch.go` `CountOrch` — orch sessions read from the prime's project directory
- `tools/tokencount/calibrate.go` `merriamTexts` — reads `CLAUDE.md` by name at the base commit
- `tools/tokencount/time_integration_test.go` — uses `-projects` with JSONL fixtures
- `tools/tokencount/calibrate_integration_test.go` — uses `-projects` with JSONL fixtures
- `internal/shuttleengine/claudeengine/usage.go` `SessionUsage`, `transcriptTokens` — the duplicate Claude usage parse
- `internal/shuttleengine/claudeengine/audit.go` `claudeProjectDirFor`, `claudeHomeDir` — Claude's projects-root and path encoding
- `internal/shuttleengine/claudeengine/command.go` — renders the launch line tokencount's Merriam read matches
- `internal/shuttleengine/claudeengine/capabilities.go` `InstructionFiles` — Claude's instruction file names
- `internal/shuttleengine/sessionusage.go` `SessionUsage`, `UsageReader` — the existing optional usage capability
- `internal/shuttleengine/engine.go` `InstructionFileLister` — the capability calibration uses
- `internal/shuttleengines/shuttleengines.go` `registry`, `New`, `SessionNamers` — the only constructor of concrete engines
- `internal/shuttleengine/seam_enforcement_test.go` `providerSeamViolations` — import scan over every production file
- `internal/websterengine/merriambase.go` `MerriamBaseOf`, `MerriamBase` — the live count calibration mirrors
