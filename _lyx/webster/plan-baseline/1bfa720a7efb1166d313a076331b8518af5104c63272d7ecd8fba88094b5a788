# Card 11 — carry-over-added-since

**Edit:**
- `internal/discussionparser#CarryOverFinding`
- `internal/discussionparser#renderCarryOver`
- `internal/discussionparser#TestWriteCarryOver_RendersConvergedAndAcceptedEntries`

**Intent:** D8 carries each `decision` finding of an accepted round into `## Open risks` naming the `Added after Discussion` entries the parent recorded since the escalation, so Plan-Write reads the finding beside the parent's answer.
`CarryOverFinding` gains `AddedSince []string`, the titles of those entries, in record order; nil means none.
`renderCarryOver` renders a finding with titles as its existing line followed by `; answered since by: <title>; <title>` inside the parentheses, each title passed through `sanitizeCarryOverField`, and a finding with none exactly as today, so every existing entry renders byte-identically.
`TestWriteCarryOver_RendersConvergedAndAcceptedEntries` gains an accepted entry holding one finding with two titles and one without, asserting both lines exactly.
Only Go writes carry-over blocks, and no parser reads the finding lines back, so no reader changes.

**ImpactSummary:** One struct field and one render line; the only writer is the Bouncer's carry-over, which sets the field from card 12 on.
