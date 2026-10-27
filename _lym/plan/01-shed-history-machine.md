# Card 1 — shed-history-machine

**Edit:**
- `internal/shedengine#Shed`
- `internal/shedengine#HistoryEntry`
- `internal/shedengine#Shed.stepLocked`
- `internal/shedengine#Shed.enter`
- `internal/shedengine#Shed.rerouteStuck`
- `internal/shedengine#TestStatus_JSONRoundTrip`

**Create:**
- `plan:internal/shedengine#TestStep_StampsTheToldMachine` -> `func TestStep_StampsTheToldMachine(t *testing.T)`

**Extends:**
- `internal/shedengine/doc.go`

**Uses:**
- `internal/shedengine#newTestShed`

**Intent:**
Decision 7: the shed stamps the machine it was told on every history entry it appends.

`Shed` gains `Machine string`, told and never read by `shedengine` itself; the empty string stamps nothing.
`HistoryEntry` gains `Machine string` with the tag `json:"machine,omitempty"`, documented like its siblings: a status file written before the field existed decodes as empty (PATTERN-tolerant-records).

Every entry the Shed itself appends carries `Machine: s.Machine`: the producer-call entry `appendHistory` builds in `stepLocked`, the skipped entry in `enter`, and the three entries in `rerouteStuck`.
`Goto` and `Relaunch` are free functions with no Shed, write entries for a goto or a resume, and stay unstamped.
A folded entry keeps the machine of the entry it folds into, which is the same shed's.

Tests:
- `TestStatus_JSONRoundTrip` gains a history entry with `Machine: "neve"` in its round-trip value, and an assertion that a status JSON whose history entry has no `machine` key decodes with an empty `Machine`.
- `TestStep_StampsTheToldMachine` is table-driven over `newTestShed`: a shed told `"neve"` whose producer returns Done persists a history entry with `Machine == "neve"`; a shed told nothing (the loom and darn case) persists an entry whose raw status file bytes hold no `"machine"` key; a skipped entry (an entry gate that holds) and a reroute entry from a told shed carry the name too.

`doc.go` gains one sentence beside the history description: a told `Shed.Machine` is stamped on each entry the shed appends, batten tells it and loom and darn do not.

**ImpactSummary:** One new told field and one tolerant JSON field; every existing shed is untold so every status file but batten's is byte-identical.

**Commit:** 1: Stamp a told machine on each shed history entry
