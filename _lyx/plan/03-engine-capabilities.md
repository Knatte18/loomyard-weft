# Card 3 — engine-capabilities

**Create:**
- `internal/shuttleengine#Requirement`
- `internal/shuttleengine#RequirementForks`
- `internal/shuttleengine#RequirementPeerMessaging`
- `internal/shuttleengine#RequirementBackgroundWake`
- `internal/shuttleengine#RequirementSupporter`
- `internal/shuttleengine#ForkAuditor`
- `internal/shuttleengine#ResumeContextRenderer`
- `internal/shuttleengine#InstructionFileLister`
- `internal/shuttleengine/claudeengine#Claude.Supports`
- `internal/shuttleengine/claudeengine#Claude.InstructionFiles`
- `internal/shuttleengine/claudeengine#Claude.ResumeContext`
- `internal/shuttleengine/claudeengine/capabilities_test.go`

**Edit:**
- `internal/shuttleengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** Declare the optional capabilities the shrunken engine seam leaves to the engines that have them (D7, D8, D9, D10), and have Claude implement them.

In `internal/shuttleengine/engine.go`, beside the other optional capabilities, each with a doc comment saying what it is and that an engine without it lacks the concern:
- `Requirement` is a closed set of what a run may require of its engine: `RequirementForks`, `RequirementPeerMessaging`, `RequirementBackgroundWake`.
- `RequirementSupporter` declares `Supports(req Requirement) bool`; an engine without it meets no requirement.
- `ForkAuditor` declares `AuditForks(sessionID, workdir string) (ForkAudit, error)` and `AuditForksIncremental(sessionID, workdir string, seenTranscripts map[string]bool) (ForkAudit, error)`, with the doc comments `Engine` carries for them today.
  `Engine` keeps both methods until card 11 moves every audit caller onto `ForkAuditor` and drops them.
- `ResumeContextRenderer` declares `ResumeContext(text string) ([]byte, error)`: it renders resume text as the provider's session-start hook output.
- `InstructionFileLister` declares `InstructionFiles() []string`: the file names the provider loads as project instructions.

In a new file `internal/shuttleengine/claudeengine/capabilities.go`, `Claude` implements the four:
`Supports` answers true for each of the three requirements and false for any other value;
`InstructionFiles` returns `CLAUDE.md` and `CLAUDE.local.md`, in that order;
`ResumeContext` returns `SessionStartContext(text)`, which stays the package function it is.
`AuditForks` and `AuditForksIncremental` already exist on `Claude`.
The file carries compile-time assertions that `*Claude` satisfies `RequirementSupporter`, `ForkAuditor`, `ResumeContextRenderer` and `InstructionFileLister`.

`internal/shuttleengine/doc.go` gains a sentence naming requirement support, resume-context rendering and instruction files as optional capabilities beside `Engine`, which an engine without them lacks.

`capabilities_test.go` adds `TestClaude_Capabilities`: `InstructionFiles` equals `[CLAUDE.md, CLAUDE.local.md]` exactly, `Supports` is true for each of the three requirements and false for `Requirement("other")`, and `ResumeContext` returns the bytes `SessionStartContext` returns for the same text.
Raise `internal/shuttleengine/claudeengine` in `test-budget.yaml` from 112 to 113.

**ImpactSummary:** Additive types in shuttleengine and new Claude methods; the edits add one package-doc sentence and raise one test-budget row.
