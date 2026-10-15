# Decision record — `lyx orch start --continue`

## Goal

After a reboot or a death of the orchestrator strand, the operator gets the same orch conversation back on asking for it with one command, `lyx orch start --continue`, instead of digging the session id out of `~/.claude/projects/` for `--adopt`.
Orch records the id of the provider session its strand is running, keeps it current across context cycles, and `--continue` resumes that session through the existing adopt path.

## Scope

In:

- Orch's persisted state records the current provider session id of the orch strand: set at every launch, and kept current by the watcher as the session's id changes.
- `lyx orch start --continue` resumes the recorded session as the orchestrator strand, exactly as `--adopt <recorded id>` would.
- Every fresh launch that saves an id other than the recorded one names the previous id's `--adopt` command in its envelope hint; a `--continue` launch never does.
- `lyx orch status` reports the recorded session id.
- The `start` help text, `internal/orchcli/doc.go` and `internal/orchengine/doc.go` describe the flag, the record and the reboot recovery.

Out:

- Making `--continue` the default for a dead strand; a plain `lyx orch start` keeps relaunching from the last handoff.
- Reed's own resume path after a server rebirth, which replays the launch's `--resume <id>` and so names the pre-clear session in clear mode (the residual `internal/orchengine/doc.go` already records).
  A strand an explicit `lyx reed resume` relaunched is live, so `--continue` refuses it; `lyx orch stop` then `--continue` recovers the post-clear session only while no watcher has ticked on the reborn strand, since the watcher records the replayed pre-clear id.
- Copying transcripts between machines (`session-sync`).

## Decisions

### D1 — `--continue` is an explicit flag, never the default

Decision: `lyx orch start` gains a boolean `--continue`.
Without it, a fresh launch chooses its prompt as today: `--handoff`, else the last completed handoff, else the start stencil.
When the state records a session id, a fresh launch's envelope `hint` names `lyx orch stop` then `lyx orch start --adopt <id>` as the way back to that conversation, with the id the state recorded before this launch written out inline, read before the launch's save replaces it.
Every fresh launch that saves an id other than the recorded one emits this hint: a launch with no prompt flag, a `--handoff` launch, and an `--adopt` launch naming another id; a `--continue` launch, and an `--adopt` launch naming the recorded id, never do.
This hint is the one place the previous id survives once D4's write has run.
When the watcher-stopping hint also applies, the envelope's hint carries both texts, the watcher-stopping one first.
Rationale: a dead or absent strand also follows `lyx orch stop`, a crashed or wedged session, or an operator who wants a clean context, where resuming the old conversation is the wrong default; the hint covers an operator who forgets the flag.

### D2 — `--continue` is `--adopt` with the recorded id

Decision: `--continue` takes the existing adopt branch of a fresh launch with the recorded session id: the same adopt prompt, the same `ResumeSessionID` on the spawn, the same shuttle resume check, the same watcher spawn and the same state reset that keeps `LastHandoff`.
The success envelope reports `prompt_source` `adopt`, as `--adopt` does, and adds a `session_id` field naming the resumed id.
Rationale: adopt is already the tested path that resumes an existing session as the orch strand; the envelope field lets the operator see which conversation came back.

### D3 — The guards `--continue` inherits and adds

Decision: `start --continue` refuses, before it removes any strand or launches anything:

- with `--adopt` or `--handoff` also given, naming the flag to drop;
- while the orchestrator strand is live, naming `lyx orch stop` first, as `--adopt` does;
- when the state records no session id (a state file an older binary wrote, or no orch ever launched from this state directory), naming the two ways forward: plain `lyx orch start` for a fresh launch, or `lyx orch start --adopt <id>` with the id `/status` shows.

The shuttle resume check then runs unchanged: it refuses a session with no transcript under the orch's working directory, which includes a session that died before its first turn, and one a live process still holds.
A launch error under `--continue` carries the same two ways forward after shuttle's own message.
A failed launch writes no state, so the recorded id stays for a retry.
Bound: `--continue` resumes only the id orch itself recorded for its own strand, either at launch or as the watcher read it from the strand's own shuttle run, so it takes over no session other than one orch launched, adopted with `--adopt`, or bound as its live orchestrator strand; the watcher's half rests on the provider naming the parent session id in a subagent's or fork's hook lines, as Open risks records.
Every guard `--adopt` has applies to it.

### D4 — Orch state records the current session id

Decision: `orchengine.State` gains a `session_id` field holding the provider session id of the strand `State.Strand` names.
`start` writes it at every fresh launch, plain or adopt, with the id the launch used; when `start` binds the state to a live strand other than the recorded one without launching, it clears the id, and the watcher's first reading of that strand's run fills it.
Shuttle's start seam returns that id beside the strand guid, since a plain launch mints it inside the engine.
The watcher keeps it current: each tick it takes shuttle's files-only session reading of the strand's run, which folds over all the run's signals, never one tick's increment; the id is the newest signal's that names one, else the run record's.
It writes the id when it differs from the recorded one, through the watcher's strand-checked save, so a watcher can never write onto a strand a concurrent `start` just bound.
Rationale: in clear mode `/clear` starts a new provider session with a new id, which the session-start signal after `/clear` names, so the launch id goes stale after the first cycle.
A record in orch's state outlives `lyx orch stop` and corpse removal.
Compact mode keeps one id, so the watcher writes nothing after the launch there.

### D5 — Shuttle exposes the session id engine-neutrally

Decision: the session reading shuttle already returns gains the session id its fold picked; orch never parses a transcript, hook payload or provider path itself.
The shuttle change is additive to the read-only reading.
Rationale: `PATTERN-shuttle-provider-seam` keeps provider specifics under `internal/shuttleengine/<engine>`; the session id is already an engine-neutral field on shuttle's session signal and run record.

### D6 — Status shows the recorded id

Decision: `lyx orch status` reports the recorded session id, empty when none is recorded.
Rationale: when `--continue` refuses or the operator wants an older session, the id is where `--adopt` needs it.

## Constraints

- `PATTERN-shuttle-provider-seam`: no Claude transcript, registry or project-directory knowledge outside `internal/shuttleengine/claudeengine`.
- `PATTERN-told-geometry`: orchengine derives no path; the state path stays told through `Paths`.
- `PATTERN-durable-vs-ephemeral-state`: the record lives in the existing never-tracked orch state file under `.lyx/orch/`; it is machine-local and survives a reboot on disk.
- `PATTERN-tolerant-records`: the new state field is optional on read; only `--continue` treats its absence as a refusal.
- The shuttle session-reading change overlaps `internal/shuttleengine`, which the running `creel-stage1-fixes` also changes; it only adds a field to the reading's result.
- A package pushed past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.

## Auto-mode assumptions

D1 through D6 and the Out bullets are auto-picks with no operator input; the support log's question ledger holds each.

## Open risks

- Claude Code's `/clear` behavior is a provider internal: if a version stops minting a new id on `/clear`, or stops naming it in the session-start hook, the watcher keeps the older id and `--continue` resumes the pre-clear conversation.
  The resume still succeeds, and the operator can `--adopt` the id `/status` shows.
- The watcher writes the id up to one poll interval after a `/clear`, and not at all while it is down with the strand alive; a power loss then leaves the pre-clear id recorded, with the same consequence.
- A resumed session near its context cap is cycled by the watcher as any session is; `--continue` does not check its size.
- A watcher bound to a strand reed's rebirth replayed records the pre-clear id, since the replayed `--resume` names it in the session-start signal; the post-clear id is then recoverable only from the provider's own session list for `--adopt`, and `lyx orch status` shows the pre-clear id.
- D4 reads the id from the newest signal of any kind, relying on Claude Code naming the parent session id in a subagent's or fork's hook lines; a version that names the subagent's own id there makes the watcher record it, and `--continue` then resumes the subagent's session.

## Acceptance criteria

- `lyx orch start --continue` with a dead or absent strand and a recorded id launches the orch strand resuming that id, with the adopt prompt, spawns the watcher, keeps `LastHandoff`, and reports `prompt_source` `adopt` and the `session_id`.
- `--continue` with `--adopt` or `--handoff` refuses, naming the flag to drop, and changes nothing.
- `--continue` with a live strand refuses, naming `lyx orch stop`, and changes nothing.
- `--continue` with no recorded id, including a state file without the field, refuses naming plain `lyx orch start` and `--adopt <id>`, and removes no strand.
- A shuttle resume-check refusal (no transcript, including a session that never took a turn, or a live holder) surfaces as the start error with the two ways forward, and the recorded id is unchanged.
- A fresh launch, plain or `--adopt`, records the id it launched with.
- Every fresh launch that saves an id other than the recorded one, whether with no prompt flag, with `--handoff` or with `--adopt` naming another id, names `lyx orch stop` then `lyx orch start --adopt <previous id>` in its hint, with that previous id, including a launch that also reports `watcher_stopping`; with none recorded, under `--continue`, or under `--adopt` naming the recorded id, no such hint.
- `start` binding a live strand other than the recorded one clears the recorded id until the watcher reads that strand's id; a strand with no shuttle run leaves it empty, so `--continue` refuses.
- The watcher records a new session id when a newer session signal names one, writes nothing when the id is unchanged, keeps the post-clear id on a later tick that reads no new signal, and never writes onto a strand another `start` has bound.
- After `lyx orch stop`, the recorded id is still there and `--continue` resumes it.
- `lyx orch status` shows the recorded id, empty when none.

## Notes for the plan writer

- `readSessionFacts` already picks the newest signal's session id with the record's as fallback; the watcher today takes the session reading only inside its idle probe, so the per-tick read is new.
