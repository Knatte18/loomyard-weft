# Card 12 — orch-start-adopt

**Edit:**
- `internal/orchcli/start.go`
- `internal/orchcli/start_test.go`
- `internal/orchengine/launch.go`
- `internal/orchengine/doc.go`
- `cmd/lyx/helptree_test.go`

**Uses:**
- `internal/shuttleengine#Run.ResumeWarning`
- `internal/orchengine#RenderAdoptPrompt`

**Intent:**
Add `lyx orch start --adopt <session-id>`: resume an existing Claude session from the prime as the orch strand, with a watcher bound to it.
The watcher needs no adopt-specific code, since it binds to the adopted transcript through the turn-end payload of this run's own settings file.

`orchengine` gains `SourceAdopt = "adopt"` beside the other prompt sources.
In `start.go`, `sessionStarter.StartSession` returns the run's resume warning beside the guid, read from `Run.ResumeWarning` in `runnerSessionStarter`;
update both test fakes in `start_test.go`, `fakeStarter` and `starterFunc`, to match.

The verb, in order:

- `--adopt` together with `--handoff` refuses before anything is locked or launched, saying the two are exclusive and to drop one.
- `--adopt` while the orch strand is live refuses naming `lyx orch stop`, the rule `--handoff` already has.
- With a dead or absent strand, the corpse is removed as today, the prompt is `RenderAdoptPrompt`, and the spec is `orchSpec` with `ResumeSessionID` set to the id.
  The engine's refusals (malformed id, no transcript under the prime's Claude project directory, a live holder) come back from `StartSession` and fail the envelope with their message intact, which names the way forward.
- State is reset with `ResetForFreshLaunch`, which keeps `LastHandoff`, and the watcher is spawned as for any relaunch.
- The launch logs at `Info` like the existing launch, naming the strand guid, the adopted session id and `promptSource` `adopt`, per Live-Substrate Spawn Observability.
- The success envelope reads `prompt_source` `adopt` and carries a `warning` field when the resume check returned one.

Bound: adopt resumes only a session recorded under the prime's own directory, never one a live process is positively identified as holding, and never kills, removes or renames any strand or process orch did not launch, including the operator's own `claude` strand.
An absent or unreadable registry lets an unconfirmed holder through, and two processes can then drive one session;
that is bounded by `--adopt` naming one id explicitly and by the warning on the log and envelope.

The flag's help text names the session id source (Claude's `/status`) and the refusal while the strand is live;
the `start` verb's `Long` text describes the adopt branch.

Tests in `start_test.go`: `--adopt` with `--handoff` refuses and starts nothing;
`--adopt` with a live strand refuses naming `lyx orch stop`;
`--adopt` with a dead strand and with no strand launches with `prompt_source` `adopt` and a spec carrying the id, `bypass`, `AllowAgentTool` and `ForkSubagents`, keeps `LastHandoff`, and spawns the watcher;
a starter warning reaches the envelope's `warning`;
a starter error fails the envelope and saves no state.
In `cmd/lyx/helptree_test.go`, assert `orch start --help` lists `--adopt`.

In the orchengine package doc, add `--adopt` to the verb list and the start-prompt order, and add an "Adopting a running session" section beside the `--handoff` migration path: read the id from `/status`, exit that session, run `lyx orch start --adopt <id>`.

**ImpactSummary:** Adds one flag and one launch branch to `orch start` and one return value to its internal starter seam, whose one production implementer and two test fakes all live in orchcli; existing start branches keep their behaviour.
