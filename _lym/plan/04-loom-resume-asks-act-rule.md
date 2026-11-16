# Card 4 — loom-resume-asks-act-rule

**Edit:**
- `internal/loomcli#loomCLI.resumeFromRunState`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#TestResumeVerb`

**Uses:**
- `internal/loomcli/doc.go`
- `internal/loomcli#runOwnerVerdict`
- `internal/loomcli#primeOfHub`
- `plan:internal/runfsm#Read`
- `plan:internal/runfsm#Act`
- `plan:internal/runfsm#Verdict.Refusal`

**Intent:**
`lym loom resume` asks the act rule before it wakes a driver (D1).
`loomCLI.resumeFromRunState` asks it right after the status read and before its `switch` on the state, through a new seam field on `loomCLI` that `loomCLI.wire` sets: `runfsm.Read` over the hub's prime (`primeOfHub`) and this worktree's slug, `Inputs.OwnedElsewhere` from `runOwnerVerdict`, and `Aside{ShedPause: true}` so a shed pause's own resume is not refused (added decision); a nil seam asks nothing.
A `wait` or `hand_over` writes the refusal envelope with `Verdict.Refusal(slug, "lym loom resume")`; a `hand_over` keeps the envelope kind `shedrun.StartOwnedElsewhereKind` so batten's InnerRun row still reads it as not retryable.
This adds a refusal only and lets nothing through that was refused before.
`TestResumeVerb` (tier0) gains rows: a cancelling run and a run owned elsewhere are refused naming their way forward, and a shed-paused run resumes.

**ImpactSummary:** `lym loom resume` refuses a run the act rule holds; the owned-elsewhere envelope kind is unchanged.

**Commit:** 4: lym loom resume asks the act rule first
