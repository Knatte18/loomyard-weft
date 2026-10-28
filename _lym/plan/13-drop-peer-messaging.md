# Card 13 — drop-peer-messaging

**Delete:**
- `internal/shuttleengine#RequirementPeerMessaging`

**Edit:**
- `internal/shuttleengine#knownRequirements`
- `internal/shuttleengine/claudeengine#Claude.Supports`
- `internal/seatengine#seatSpec`
- `internal/seatengine#Seat`
- `internal/shuttleengine#newRequiresRig`
- `internal/shuttleengine#TestRunner_Requires`
- `internal/shuttleengine/claudeengine#TestClaude_Capabilities`
- `internal/seatengine#TestSeatPrompts_ValuesPromptsAndSpecs`
- `internal/seatengine/doc.go`

**Uses:**
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine/doc.go`

**Intent:**
`RequirementPeerMessaging` goes (D7): after card 12 no role messages a peer through an engine's own tool, so nothing requires it.
The constant leaves `engine.go`'s requirement block, `knownRequirements` becomes `{RequirementForks, RequirementBackgroundWake}`, `Claude.Supports` stops naming it, and `seatSpec` adds no requirement for a table of more than one seat, so a seat's spec carries only the seat's own `Requires`; its doc comment and the `Seat.Requires` field comment drop the peer-messaging clause.
What stays Claude-only is declared as before: in-session forks by `ForkSubagents` (`RequirementForks`) and background wake by `RequirementBackgroundWake`.

Tests: `newRequiresRig`'s full engine supports forks and background wake; `TestRunner_Requires`' expected known-requirement lists read `forks, background-wake`, its rows that required peer messaging require background wake instead, and its unknown-requirement row at start takes `"peer-messaging"` in place of `"telepathy"`, so the removed name is refused as unknown, naming the known ones, with no new row; `TestClaude_Capabilities` drops the peer-messaging row; `TestSeatPrompts_ValuesPromptsAndSpecs` asserts a multi-seat table's seat spec carries only the seat's own `Requires`.
`internal/seatengine/doc.go` drops the clause that the spec adds peer messaging, and its opening says the advisors talk to the chair through creel mail.

**ImpactSummary:** One requirement constant and its four production references go; a multi-seat table no longer demands peer messaging of its engine, and the five tests that named it change with it.
