# Card 16 — webster's begin-batch and recover-batch refuse under a pause and claim Master

**Create:**
- `plan:internal/webstercli#websterCLI.refuseCoveredBatch` -> `func (c *websterCLI) refuseCoveredBatch(verb, batch string) string`
- `plan:internal/webstercli#TestBatchVerbs_RefuseUnderAPause` -> `func TestBatchVerbs_RefuseUnderAPause(t *testing.T)`
- `plan:internal/webstercli#envMasterStrand` -> `func envMasterStrand(resolve func(name string) (string, error)) func() (name, guid string, err error)`

**Edit:**
- `internal/webstercli#websterCLI`
- `internal/webstercli#websterCLI.wireHub`
- `internal/webstercli#websterCLI.wireStandalone`
- `internal/webstercli#websterCLI.beginBatchCmd`
- `internal/webstercli#websterCLI.recoverBatchCmd`

**Uses:**
- `internal/pauserecord#ClaimSession`
- `internal/pauserecord#ReadCoverage`
- `internal/webstercli#newVerbsFixture`
- `internal/webstercli#verbsFixture`
- `internal/webstercli#verbsFixture.initState`

**Intent:**
D8.
`websterCLI` gains the seam field `masterStrand func() (name, guid string, err error)`, which `wireHub` and `wireStandalone` set to `envMasterStrand(reedEngine.ResolveStrandGUID)` over the reed engine each builds.
`envMasterStrand` returns the function that reads the verb's own `agentname.StrandNameEnv` and resolves its GUID through `resolve`; an unset or empty name is an error naming the variable, and a resolve error is returned as is.
`refuseCoveredBatch` returns the refusal message, or the empty string to go on, and reads `c.pause`, the geometry card 6 stores:
a zero geometry goes on;
`pauserecord.ReadCoverage(c.pause.RecordPath, c.pause.Slug, now)` failing logs one Warn naming the record and goes on, as loomcli's `refuseCovered` does;
an uncovered slug goes on.
Under a covering entry it calls `masterStrand`, and on success `pauserecord.ClaimSession(c.pause.RecordPath, c.pause.Slug, Session{Strand: name, GUID: guid, Slug: c.pause.Slug}, now)`, so `lyx tender resume` gives Master its resume line.
A claim that answers uncovered, the entry lifted between the read and the claim, records nothing and goes on, since no pause holds the pair any more and no resume line would come.
A claim that answers covered returns:
`webster: <the pause> holds this pair, so <verb> starts no new batch; way forward: end your turn now; a running batch finishes, and once the resume line arrives run "lyx webster <verb> <batch>" again`.
A failed GUID lookup, or a claim that fails, records nothing, logs a Warn naming the strand, and returns:
`webster: <the pause> holds this pair, so <verb> starts no new batch, and no resume line will come; way forward: end your turn now, then run "lyx webster <verb> <batch>" again once "lyx tender status" shows no pause`.
`<the pause>` is `a hub-wide pause` or `the pause of <slug>`.
Neither text names `lyx resume`, which would lift the operator's pause, nor any denied form (`PATTERN-no-denied-recovery`), and each ends in its `way forward:` clause (`PATTERN-refusal-way-forward`).
`beginBatchCmd` and `recoverBatchCmd` call it right after `clihelp.ShouldAbort`, before the conflict-record removal, the plan parse and the state-mutation lease, with `args[0]` as the batch, and print a non-empty answer with `output.Err` and exit 1, so a refused call changes no state.
`record-batch`, `await-batch` and `status` take no such check, so Master can record a finished batch under a pause.
Both verbs' `Long` text names the refusal.
Bound: the refusal only defers the verb.

`TestBatchVerbs_RefuseUnderAPause` drives the verbs through `newVerbsFixture` with `fx.CLI.pause` and the runner told a temp record and `fx.CLI.masterStrand` stubbed:
`begin-batch 1` and `recover-batch 1` under a live hub entry exit 1 with the first message, leave `state.json` byte-identical, and record Master's name and GUID in the entry;
`begin-batch 1` and `recover-batch 1` whose `masterStrand` fails exit 1 with the second message, whose way forward opens with `end your turn now`, record nothing, and log one Warn naming the strand;
`begin-batch 1` whose record turns undecodable after the coverage read (a `masterStrand` stub that corrupts it before answering) exits 1 with the second message and records nothing;
`begin-batch 1` whose entry is resumed after the coverage read (a `masterStrand` stub that resumes it before answering) goes on and begins the batch;
`begin-batch 1` over an undecodable record succeeds and logs one Warn;
`record-batch`, `await-batch` and `status` under the live entry are not refused by the pause;
no message contains `lyx resume`.
The same test calls `envMasterStrand` with `agentname.StrandNameEnv` set empty through `t.Setenv`, so it does not run in parallel, and gets the error naming the variable, then with it set and a resolve stub, and gets the name and the stub's GUID.

**ImpactSummary:** two webster role verbs gain one up-front refusal through one helper and one new seam filled by both wiring paths; the other verbs and webster's engine are unchanged.
