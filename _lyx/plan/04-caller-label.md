# Card 4 — caller-label

**Create:**
- `internal/agentname#CallerOperator`
- `internal/agentname#CallerLabel`
- `internal/agentname#TestCallerLabel`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Add the one classification of a caller's strand name into the label the shed history records (card 5), in `internal/agentname`, the sole parser of agent names, which imports the standard library only.

`CallerLabel` maps a strand name, the value of `StrandNameEnv`, to a label: `CallerOperator` for an empty strand; `RoleOrch` for a strand whose role `MatchesRole` the orch's; `RoleDriver` for one whose role `MatchesRole` the driver's, the loom LLM run driver; and the role itself for any other strand.
A strand that does not parse is never refused: its label is the text after its last `:`, or the whole strand when it has none, the raw role.
The label is for reports only; no Go decision reads it, which is the bound on a caller choosing its own strand.
The CLI layer calls it (card 6), so `shedverbs` and `shedengine` take the label as a plain string and gain no import.

`TestCallerLabel` in `internal/agentname/agentname_test.go` is a table: an empty strand reads `operator`; `lyx:orch` reads `orch`; `lyx:orch-2` reads `orch`; `lyx:my-task:driver` reads `driver`; `lyx:my-task:driver-2` reads `driver`; `lyx:my-task:alignment` reads `alignment`; `garbage` reads `garbage`; `Bad:Role` reads `Role`.
`CallerLabel`'s doc comment states the mapping and the bound; `cmd/lyx/testdata/test-budget.yaml` raises `internal/agentname` to 9.

**ImpactSummary:** one new pure function and constant in a stdlib-only leaf; no caller yet.

**Commit:** 4: Classify a caller's strand into a history label
