# Card 3 — start-continue

**Create:**
- `plan:internal/orchcli#startedSession` -> `type startedSession struct`
- `plan:internal/orchcli#previousSessionHint` -> `func previousSessionHint(id string) string`
- `plan:internal/orchcli#withContinueWaysForward` -> `func withContinueWaysForward(err error) error`

**Edit:**
- `internal/orchcli#sessionStarter`
- `internal/orchcli#runnerSessionStarter.StartSession` -> `func (s runnerSessionStarter) StartSession(spec shuttleengine.Spec) (startedSession, error)`
- `internal/orchcli#orchCLI.startCmd`
- `internal/orchcli#orchCLI.adoptStrand`
- `internal/orchcli#fakeStarter`
- `internal/orchcli#fakeStarter.StartSession` -> `func (f *fakeStarter) StartSession(spec shuttleengine.Spec) (startedSession, error)`
- `internal/orchcli#starterFunc`
- `internal/orchcli#starterFunc.StartSession` -> `func (f starterFunc) StartSession(spec shuttleengine.Spec) (startedSession, error)`
- `internal/orchcli#newStartHarness`
- `internal/orchcli#TestStart_DeadStrandRemovedBeforeStart`
- `internal/orchcli#TestStart_LiveStrand`
- `internal/orchcli#TestStart_Refusals`
- `internal/orchcli#TestStart_StarterErrorFailsAndSavesNoState`
- `internal/orchcli#TestStart_AdoptLaunchesResumeSpec`
- `internal/orchcli/doc.go`
- `internal/orchengine/doc.go`

**Uses:**
- `internal/orchcli#startHarness.run`
- `internal/orchcli#startHarness.state`
- `internal/orchcli#stoppingWatcherHint`
- `internal/orchcli#runVerb`
- `internal/orchcli#orchCLI.stopCmd`

**Intent:** `lyx orch start --continue` resumes the session id orch recorded, through the existing adopt branch (D1, D2, D3), and every fresh launch records the id it launched with (D4).

The start seam returns the session id:

- `startedSession` is a struct with fields `GUID`, `SessionID` and `Warning`, all strings: the strand guid, the provider session id the launch used (minted for a fresh launch, the resumed id for a resume) and the resume check's warning, empty when there is none.
  Named fields, not three positional strings, so no adapter or double can swap the id and the warning unseen.
- The `sessionStarter` interface's `StartSession` method returns `(startedSession, error)`.
- `runnerSessionStarter.StartSession` returns `startedSession{GUID: run.StrandGUID(), SessionID: run.SessionID(), Warning: run.ResumeWarning()}` (card 1's accessor).

`startCmd` gains a boolean `--continue` flag, help "resume the orch session recorded at the last launch or cycle as the orchestrator strand; exclusive with --adopt and --handoff, refused while the strand is live, and with no session recorded".
Its guards refuse before `start` removes the orch strand or launches anything, so a refusal changes neither the orch strand nor the state.
As for `--adopt` today, the live and no-id checks run after `reedUp` and `ensureRunsStrand`, which may add or replace the `runs` strand; the exclusivity checks run before both.

- `--continue` with `--adopt`: `orch: --continue and --adopt are exclusive; drop --adopt`; with `--handoff`: `orch: --continue and --handoff are exclusive; drop --handoff`.
  Both sit beside today's `--adopt`/`--handoff` exclusivity check, ahead of the lock.
- `--continue` while the orchestrator strand is live: `orch: --continue applies to a fresh launch only, but the orchestrator strand <guid> is live -- run `lyx orch stop` first`, beside the `--adopt` live check after `DecideStart`.
  The input is a live strand named `orch`, whatever the recorded id.
- `--continue` when `st.SessionID` is empty, checked right after the live check and before the `StartRelaunch` branch's `RemoveStrand`: `withContinueWaysForward(errors.New("orch: --continue found no recorded orch session id"))`.
  The input is a dead or absent strand with a state file lacking `session_id`, from an older binary or with no state file at all.
- `withContinueWaysForward(err)` returns `fmt.Errorf("%w; run `lyx orch start` for a fresh launch, or `lyx orch start --adopt <id>` with the id `/status` shows in the session", err)`.

Past the guards, `--continue` sets the resume id to `st.SessionID` and takes the adopt branch unchanged: `RenderAdoptPrompt`, `SourceAdopt`, `spec.ResumeSessionID`, shuttle's resume check inside `StartSession`, the watcher spawn and the state reset that keeps `LastHandoff`.
A `StartSession` error under `--continue` is returned through `withContinueWaysForward`, after shuttle's own message; a failed launch saves no state, so the recorded id stays for a retry.
Bound: `--continue` resumes only `st.SessionID`, which only `start` (the id its own launch used, or one an operator gave `--adopt`) and the watcher (read from the bound strand's own shuttle run) write, so, within the reliance below, it takes over no session orch did not launch, adopt or bind; shuttle's resume check still refuses a session with no transcript under the prime or one a live process holds.
The watcher's half rests on the provider naming the parent session id in a subagent's or fork's hook lines, since the orch strand runs with `ForkSubagents` and the claude engine takes each hook line's `session_id` as given, so a provider that names the subagent's own id there makes `--continue` resume the subagent's session.

The fresh-launch save sets the id: `next := orchengine.ResetForFreshLaunch(st, started.GUID, now)`, then `next.SessionID = started.SessionID`, then `SaveState`.
A launch whose engine names no id records an empty id on purpose, since the record names the session the strand runs, and the hint below then keeps the previous id.
The adopt log line also fires under `--continue`, and the launch log line carries the session id.
When `st.SessionID` is non-empty and differs from the launched id, the envelope's hint carries `previousSessionHint(st.SessionID)`, read from the state loaded before the save, which returns `the previous orch session <id> is still resumable: run "lyx orch stop" then "lyx orch start --adopt <id>"`.
That covers a plain launch, a `--handoff` launch and an `--adopt` naming another id; `--continue` and an `--adopt` naming the recorded id launch with the recorded id, so they carry none.
When the watcher-stopping hint also applies, the hint is `stoppingWatcherHint + "; " + previousSessionHint(...)`, the watcher-stopping text first.
Under `--continue` the success envelope adds `session_id`, the resumed id; `prompt_source` stays `adopt`.

`adoptStrand` binding a live strand other than the recorded one saves the reset state with `SessionID` cleared, so the watcher's first reading of that strand's run fills it, and a strand with no shuttle run leaves it empty for `--continue` to refuse.

`startCmd`'s `Long` text gains a paragraph on `--continue`: it resumes the session orch recorded for its strand at launch and after each `/clear`, the reboot recovery after `lyx orch stop` or a dead strand, refused alongside `--adopt` or `--handoff`, while the strand is live, or with no session recorded; and that a fresh launch replacing a recorded session names it in the envelope hint.

Docs: `internal/orchcli/doc.go`'s role-file paragraph says skills load on every launch path, "`--adopt` and `--continue` included", and gains a sentence that `start` records the orch session's id in orch's state, which `lyx orch start --continue` resumes after a reboot or a dead strand, pointing at the internal/orchengine package documentation's `Continuing after a reboot` section.
`internal/orchengine/doc.go`: the `start` bullet of `# Verbs, and where they run` names `--continue`; `# Start prompt order` item 1 covers `--adopt` and `--continue`; a new `# Continuing after a reboot` section after `# Adopting a running session` says `lyx orch start --continue` resumes the recorded session after a reboot or a dead strand, that a plain `start` still relaunches from the last handoff, its refusals, and that `lyx orch status` shows the recorded id; the reed-rebirth residual adds that the watcher bound to a reborn strand records the replayed pre-clear id, so `--continue` then resumes the older context too and the post-clear id is recoverable only from the provider's session list for `--adopt`.

Tests:

- `fakeStarter` gains `sessionID string`; its `StartSession` answers `startedSession{GUID: guid, SessionID: spec.ResumeSessionID when set, else sessionID, Warning: warning}`.
  `newStartHarness` builds it with `sessionID: "minted-id"`.
  `starterFunc` becomes `func(shuttleengine.Spec) (startedSession, error)` and `TestStart_DeadStrandRemovedBeforeStart`'s closure returns `startedSession{GUID: "new-guid", SessionID: "minted-id"}, nil`.
- `TestStart_Refusals`: each case gains an optional `seed func(h *startHarness)` that writes the state file, and every row also asserts no `RemoveStrand` call and a state file byte-identical to the seeded one (or still absent).
  New rows: "continue with adopt" (`--continue --adopt <id>`, want `drop --adopt`), "continue with a handoff" (`--continue --handoff <file>`, want `drop --handoff`), "continue with a live strand" (live `g1`, state recording `rec-id`, want `lyx orch stop`), "continue with no recorded id" (dead strand `corpse`, a raw state file `{"strand":"corpse","phase":"idle"}` written with `os.WriteFile` to `paths.StatePath`, so the `session_id` key is absent as an older binary writes it; want `lyx orch start` and `--adopt <id>`), which also pins that the corpse is not removed, and "continue with no state file" (no strand, no state; same want).
- `TestStart_StarterErrorFailsAndSavesNoState`: a row "continue" seeds `{SessionID: "rec-id"}`, runs `--continue` with the starter failing `no transcript for that session`, and wants the error to carry that text and `lyx orch start --adopt <id>`; the saved-state assertion becomes "the seeded state, unchanged", so this row's id stays `rec-id`.
- `TestStart_LiveStrand`: a row "a live strand other than the recorded one clears the session id" seeds `{Strand: "old", SessionID: "old-id"}` with no watcher, wants `spawned-watcher`, strand `g1` and an empty id; the seeded stopping rows record `SessionID: "g1-id"` on strand `g1` and want it kept.
- `TestStart_AdoptLaunchesResumeSpec` becomes the table for every fresh launch's resume id, prompt, recorded id and hint.
  Each case gains `args`, a seeded recorded id, the starter's `sessionID` when not `minted-id`, a `stopping` flag that seeds `WatcherStopping` and holds the watch lock for the whole run, and wants: `ResumeSessionID`, `prompt_source`, the saved `SessionID`, the exact `hint` (absent when empty), and the `session_id` key (absent when empty).
  Every row keeps today's asserts of `action` `relaunched`, one start and one spawn and `LastHandoff` kept; the adopt and continue rows also assert `spec.Prompt` equals `orchengine.RenderAdoptPrompt(h.cli.stencilsDir, h.cli.paths.RolePath)`.
  The existing three `--adopt` rows keep their strands and warnings, record no id and want `adoptTestSessionID` saved, no hint and no `session_id` key.
  New rows:
  - "plain launch with none recorded": no args; want resume id empty, source `fresh`, saved `minted-id`, no hint.
  - "plain launch replaces a recorded id": recorded `old-id`; want saved `minted-id`, hint `previousSessionHint("old-id")`.
  - "handoff launch replaces a recorded id": `--handoff <file>`, recorded `old-id`; want source `flag`, saved `minted-id`, the same hint.
  - "a launch naming no id records none and keeps the previous in the hint": recorded `old-id`, starter `sessionID` empty; want saved empty, hint for `old-id`.
  - "adopt naming another id": `--adopt adoptTestSessionID`, recorded `old-id`; want saved `adoptTestSessionID`, hint for `old-id`.
  - "adopt naming the recorded id": recorded `adoptTestSessionID`; want it saved, no hint.
  - "continue after stop resumes the recorded id": a live strand `g1` recorded with `adoptTestSessionID`; the row first runs `runVerb(t, h.cli.stopCmd())`, then sets `h.strands.strands` to nil as reed reports after the removal, then runs `--continue`; want resume id, saved id and `session_id` all `adoptTestSessionID`, source `adopt`, the adopt prompt and no hint.
  - "a stopping watcher and a replaced id carry both hints": recorded `old-id`, `stopping`; want saved `minted-id`, `watcher_stopping` true and hint `stoppingWatcherHint + "; " + previousSessionHint("old-id")`.

**ImpactSummary:** `sessionStarter` is orchcli-private with one production adapter and two test doubles, all re-signed here onto `startedSession`; `startCmd` gains one flag and its guards, `adoptStrand` clears the id on a rebind; no other package calls the seam, and no new top-level test is added.
