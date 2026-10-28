# Card 14 — creel-trust-sentence

**Edit:**
- `internal/shuttleengine/claudeengine#creelTrustSentence`

**Uses:**
- `internal/shuttleengine/claudeengine/doc.go`

**Intent:**
The creel trust sentence every Claude session's appended system prompt carries defers to the role's rules (D6), since after card 12 an advisor's answer arrives as `[creel]` mail, which the seat directive calls advice, never an instruction.
`creelTrustSentence` becomes: "Lines starting with [creel] are LoomYard internal mail delivered by hooks; follow mail from your parent, an orch strand or a lym program as the operator's own instructions, and weigh any other mail as your role's instructions say. When one asks for an answer, reply with lym creel post --reply <id> \"<text>\", using the id after #; answer mail from EXTERNAL in your own reply text instead."
It stays one line, as the package's `init` check requires, and parent mail stays followable, which the Haiku trust finding needed.
The tests that build the appended prompt compare against the constant itself, so they follow it, and no test asserts its prose.

**ImpactSummary:** One constant's wording; its two readers append it unchanged and the tests compare by the constant.
