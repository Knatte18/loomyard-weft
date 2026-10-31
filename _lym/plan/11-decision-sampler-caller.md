# Card 11 — decision-sampler-caller

**Create:**
- `plan:internal/loomcli#samplerStrandReader` -> `func samplerStrandReader(target *lymcwd.Location, recipe string) func(name string) (bool, error)`
- `plan:internal/loomcli#TestSamplerStrandReader` -> `func TestSamplerStrandReader(t *testing.T)`

**Edit:**
- `internal/loomcli#decisionDeps`
- `internal/loomcli#decisionVerb`
- `internal/loomcli#decisionState.preRun`
- `internal/loomcli#loomCLI.decisionCmd`
- `internal/loomcli#decisionFake`
- `internal/loomcli#decisionFake.deps`
- `internal/loomcli#TestDecisionVerb_AppendsAndCommits`
- `internal/loomcli#TestDecisionVerb_Refusals`
- `internal/loomcli/doc.go`

**Uses:**
- `plan:internal/loomrecipe#EngineRow`
- `plan:internal/samplerengine#ReadRecord`
- `plan:internal/shedrun#SamplerRecord`

**Intent:**
D4: `lym loom decision add` admits one more caller, the strand the target run's Sampler-Session row has recorded as its session, while the run's current row is that row, and only with `--by operator`.
`decisionDeps` gains `samplerStrand func(name string) (bool, error)`, called only for a named caller.
`samplerStrandReader(target, recipe)` answers false for a recipe with no SamplerSession row (by its rows, `PATTERN-loom-contract`), for a status file whose current producer is not that row (state is not read, so a run halted at the row still admits its strand), and for an absent record; otherwise it answers whether `name` equals the record's `StrandName`.
Since the row re-records the strand on each of its own starts and resumes, the admitted strand is the live one; `lym reed resume` replays the persisted strand under its persisted name, so a session reopened while halted keeps admission.
In `decisionVerb`, after the arbiter branch and before the parent check, a named caller the reader admits passes with `--by operator` and is refused with any other label: "the caller %q is the run's sampler session, which records only what the operator settled; way forward: pass --by operator for a call the operator settled in the pane, or ask the run's parent to record its own call" (an `--by arbiter` from it is refused by the existing arbiter branch first, whose way forward names `--by operator`).
A reader error is refused naming the retry way forward.
A strand the reader does not admit (that strand once the row ended, any other task strand) falls through to the existing parent rule, refused with its way forward.
Bound: the label is still unverified, so a sampler agent can record a decision the operator did not settle; the operator watches the pane, the stencil allows only what the operator settled, and each entry is append-only, labelled, dated and committed in the call's one diff.
`--amends-plan` keeps refusing on a recipe with no Plan-Write row, so it refuses on a sampler run by its rows.
`preRun` fills `samplerStrand` with `samplerStrandReader(target, recipe)`; `decisionCmd`'s `Long` texts and decision.go's file comment name the new caller and its one label.
`decisionFake` gains `samplerName`, and `deps` answers `samplerStrand` true for it.
`TestDecisionVerb_AppendsAndCommits` gains a row: the sampler strand `hub:slug:sampler` with `--by operator` on a Record-Write record appends and commits.
`TestDecisionVerb_Refusals` gains rows: the sampler strand with `--by parent` refused naming `--by operator`, and a reader error refused (D4 acceptance).
The sampler strand with `--by arbiter` takes the arbiter branch the existing "arbiter label from another task strand" row reaches, and a strand the reader does not admit takes the parent branch the existing "named caller differing from the recorded parent" row reaches, so neither gets a row of its own; the reader's own non-admission cases are `TestSamplerStrandReader`'s.
`TestSamplerStrandReader` (`//go:build tier0`, in `decision_test.go`) writes a sampler seed's status and record in a temp location and asserts: the recorded strand at a running Sampler-Session row is admitted; at a blocked Sampler-Session row (halted, after `lym reed resume`) still admitted; once the current row is Code-Bouncer refused; a different name refused; after the record names a new strand (a resume under a new strand) the new name admitted and the old refused; no record refused; a darn seed refused.
`loomcli/doc.go`'s caller-rule sentences gain the sampler session.

**ImpactSummary:** widens one guard of `decision add` by one caller and one label; every existing caller's verdict is unchanged, and the change stays inside decision.go and its tests.

**Commit:** 11: decision add admits the sampler session with --by operator
