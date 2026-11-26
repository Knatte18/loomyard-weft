# Coordinating the implementer forks for the nested config keys plan

Card 11 was one batch covering the Go messages of every other module.
The harness reported about 618k `subagent_tokens` and 470 tool uses for its fork, and the operator flagged that size as too large.
Cards of this breadth want splitting in the plan.

Card 11 also finished `done` while its fork reported bare key names left outside its target packages:
`gatecli`, `batcher`, `battencli`, `beamcli`, `webstercli`, `hibernatecli`, `reportcli`, `preflightshed`,
`weaverengine` (`ResolveLadder`, `settingRefusal`), `internal/sheddriver/tackler.go`, `shedarbiter`,
`internal/burlerengine/fix.go` and `internal/loomshed/plancontext.go`.
No later card was seen to claim them, so review should check whether they are still bare.
