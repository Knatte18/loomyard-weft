# Decision record — bugfix-sessions

## Goal

Fix the session-lifecycle, notice and halt bugs the board entry `bugfix-sessions` collects, each with a regression test that drives the real CLI wiring, so that typed prompts are submitted, superseded sessions never run beside their successors, halts and step handoffs report only what happened, the orch watcher reloads only after a fresh compaction and survives a restart, a Burler review with broken YAML is repaired in its own session, and no start verb moves a tmux client.

## Scope

In scope, one section per reported bug:

- `producer-session-handoff` (#395): a typed prompt left unsubmitted in a Claude Code input box.
- #399: a reflection session spawned on an `escalation-to-human` halt.
- #400 and the Webster recovery strand: a halted producer's strand, or a finished recovery strand, left alive beside its successor.
- `false-crash-resume` (#396, #392): a `loom-crash-resume` note on a clean step handoff, and a note that hides its evidence and truncates history silently.
- `orch-watcher-stale-reload` (#390), its restart race, and #393: a watcher that reloads after an old compaction, a reload that retypes skills a compaction keeps, `lyx orch start` racing a dying watcher, and a long-lived orch session that cannot see a skill deployed after it started.
- `burler-yaml-frontmatter-repair` (#386): a Burler round that fails the run on an invalid review frontmatter.
- `orch-resume-without-attach`, terminal half only: `lyx loom start` and `lyx orch start` never attach or switch a tmux client.

Out of scope:

- The orch resume verb and the role stencil's switch to it (stays on the `orch-resume-without-attach` note).
- One channel per halt notice, event-driven batten notices (`orch-notice-channel`).
- Operator shortcuts that combine start and attach (`operator-shortcuts`).
- Any new verb, config key or feature other than D12's self-check verb, which `PATTERN-gate-self-check-parity` requires; a reviewer treats any other extension as a scope finding.

## Decisions

### D1 — Submission is confirmed, not only appearance (#395)

Decision: shuttle's verified send path confirms that a delivered text was submitted, not only that it appeared in the pane.
After the text is seen in the pane, shuttle waits a provider-owned settle after the Enter and reads the input box only after that settle; while the box still holds the text, shuttle presses Enter again, up to two extra Enters, each followed by the same settle before the next read, then fails the send with an error naming the pending input.
The settle is a provider-owned constant long enough for the box to be redrawn, so a send whose Enter landed is not read as still pending; it goes through the same injectable sleep as `sendVerified`'s polling, so a test shortens it.
Reading the input box is a provider capability that lives in `claudeengine`, beside the existing idle probe, under `PATTERN-shuttle-provider-seam`.
Matching rule: the needle is the one `sendVerified` builds, the sent text with line wrapping and whitespace removed and case folded, cut at the first 48 characters.
The box counts as holding the sent text when its content, normalized the same way, contains the needle and, for a text shorter than 48 normalized characters, equals the whole normalized text, so a short text inside a longer draft does not match.
Anything else, a collapsed paste placeholder included, counts as not holding it, and the send then falls back to the appearance-only check (R5).
Bound: an extra Enter is sent only when the provider reports the input box holding the sent text, so it never lands on an empty box, a running turn or a draft that lacks the needle; a draft that contains the needle of a text of 48 or more characters is indistinguishable from the sent text and is submitted by the Enter.
At most two extra Enters per send.
A provider without the capability keeps today's appearance-only check.
Rationale: every typed delivery, whatever its caller, reaches the pane through shuttle's verified send, so one fix there covers all of them; checking the box detects a lost Enter whatever its cause.

### D2 — Text and Enter are sent as two paced steps (#395)

Decision: `claudeengine`'s key sequences that type text and submit it (`ComposeSend`, `ModelSwitchSequence`, and any other text-plus-submit sequence) type the text unsubmitted, settle for a provider-owned delay, then send Enter as its own key step.
`reedengine.SendText` keeps its transport contract unchanged.
Rationale: Claude Code reads a fast character burst as a paste and swallows an Enter that arrives inside it; the settle moves the Enter out of the burst, and the pacing is a provider fact, so it lives in `claudeengine`.

### D3 — An escalation halt spawns no reflection (#399)

Decision: a halt classified `escalation-to-human` writes its halt note and spawns no reflection session; the envelope's `friction` key reports skipped.
Every other halt kind reflects as today.
The terminal Friction-Reflect row of a resumed run still reads the note, so `PATTERN-friction-capture` holds: the note is written and is not archived before a reflection covers it.
Bound: only the `escalation-to-human` kind skips, and the skipped reflection is the only thing lost; the note stays on disk unarchived until that row covers it.
Rationale: the producer's own session holds the open question; a second session reflecting on it looks like the run moving on and finds nothing but a process signal.

### D4 — A re-run producer removes its predecessor's strand (#400)

Decision: when a producer row starts a fresh shuttle run after its attach probe found nothing attachable, shuttle first removes the live strand of every earlier run of the same output-file set, the set the attach probe already matches on, whose run the probe judged respawn-eligible.
Removal goes through reed's strand removal and is logged.
Bound: it removes only strands whose run record declares the exact output-file set of the new run and whose run the probe judged respawn-eligible, so it never touches another producer's strand, a parallel same-role strand with other outputs, or a run the probe would attach to; a start the probe refuses removes nothing.
A failed removal fails the start with an error naming the strand and `lyx reed remove`, rather than spawning beside it.
Rationale: the attach probe already decides "same producer, same outputs"; extending its respawn branch closes the gap for every shuttle-backed row with one rule, and re-running a producer is the operator's decision to supersede the halted session.

### D5 — Webster removes a finished or superseded recovery strand

Decision: `recover-batch` removes its recovery strand on every terminal outcome whose strand is still live, the failed-batch path included, and `webster reset` removes every live recovery strand of the batches it resets; each removal is logged.
Bound: only strands Webster's own batch state names as its recovery strand; an implementer or master strand is untouched by this decision.
Rationale: the rule from D4, applied to Webster's own strands, which do not go through the producer attach probe.

### D6 — A clean step handoff is never a crash-resume (#396, #392)

Decision: the entry observation never reports a crash-resume for a run whose previous step completed and was handed off cleanly, which is the reported case: last history entry `Webster / done`, state `running`, the next producer not yet started.
The plan writer finds the cause by reproducing that sequence through the CLI (a Webster producer completing, then the next `step`) and fixes it at the cause; the voucher mechanism stays the means of telling a handoff from a death unless the cause shows it cannot.
Bound: a driver that dies before it records the handoff voucher, mid-producer or between a finished step and the voucher, leaving none matching the observation, is still reported, and any replacement of the voucher mechanism keeps that; the acceptance criteria for this decision are the whole contract.
Rationale: a false crash note sends the reader hunting for a crash that never happened.

### D7 — The crash-resume note shows its evidence

Decision: the `loom-crash-resume` note records why it fired: whether the run lock was held, whether a handoff voucher was found and, when found, its recorded history length and state beside the observed ones.
It lists every history entry or, when it lists only the trailing rows, says how many it left out.
Rationale: the reader must be able to tell a real crash from a false positive from the note alone.

### D8 — The orch watcher reloads only after a fresh compaction (#390)

Decision: an auto-compaction boundary enters the reload sequence only when exactly one turn end follows it in the transcript the watcher reads, and that turn end is the one being read, the newest turn end among the events read on the tick.
A boundary with more than one turn end after it, including two read in one tick and one found by a restarted watcher on an old cursor, moves `State.CompactionBaseline` past it with a log entry and no reload.
A boundary with one later turn end that is not the one being read, or with none, types nothing and leaves the baseline, so a later turn end evaluates it again.
Rationale: a reload typed after turns that ran without it no longer answers the compaction it was meant for, and a watcher restarted on an old cursor otherwise types a reload into a session working normally.

### D9 — A reload after a compaction retypes no skills (#390)

Decision: the reload sequence entered after a compaction, watcher-run or automatic, skips the skills step and the retry step and goes on to the pointer step; only the reload after `/clear` runs them.
The premise is A2, taken from the board entry; it supersedes the present statement in `internal/orchengine/doc.go`'s "The reload sequence" section that a compaction loses the session's skills, and that section is updated to say so.
Bound: only the compaction entries skip the skills step, `/clear` keeps it, and the pointer step still has the session read its role file; a session that did lose a skill misses it until the next `/clear` or restart (R4).
Rationale: Claude Code re-injects the skills a session invoked after a compaction, so typing them again costs a turn and changes nothing; `/clear` loses them.

### D10 — The orch session reloads plugins before every reload sequence (#393)

Decision: before every reload sequence, after `/clear`, after a watcher-run compaction and after an auto-compaction, the watcher types Claude Code's `/reload-plugins` as a provider key sequence from `claudeengine`, under the same idle-probe rule as every other typing.
It is idempotent, so a restarted watcher may type it again.
It is the first step of the sequence, before the skills step after `/clear` and before the pointer step after a compaction, and it is persisted as a step of its own so a restart resumes at it.
The same "The reload sequence" section of `internal/orchengine/doc.go` describes this step.
Bound: it is typed only on a tick whose idle probe passed, ends no turn and starts none, so the watcher waits for the next passing idle probe before the next step.
Rationale: Claude Code builds its skill list at process start, and `/reload-plugins` rebuilds the model-facing list the Skill tool reads; since `skills-in-one-turn` every skill loads through the Skill tool, so a skill deployed after the session started becomes loadable at the next reload without restarting the process.

### D11 — `lyx orch start` waits for a stopping watcher (#390 restart race)

Decision: a watcher that receives SIGINT or SIGTERM records that it is stopping in the orch state before it finishes its tick and releases `watch.lock`; a watcher clears that record when it starts.
`lyx orch start` that finds the lock held while the state records the watcher stopping waits for the lock to free, bounded by a fixed wait of a few watcher poll intervals, then decides as usual.
Bound: the wait happens only while the stopping record is present; past the bound, start reports the watcher live, as today, and the envelope says it is stopping and names `lyx orch start` to run again once the watcher has exited.
Rationale: a `kill` followed by `start` otherwise reads the dying watcher as live and spawns none.

### D12 — A Burler review with invalid frontmatter is repaired in its own session (#386)

Decision: `burlerengine` adds one gate entry to every round's shuttle gate that parses the review file with `ParseReview`, the same function that reads the verdict afterwards, so an unparseable review is re-prompted to the reviewer in the same session with the parse error and its quoting hint, inside the shuttle gate's existing attempt budget.
The parse after the gate stays; it fails the round only when the gate's budget is spent.
`PATTERN-gate-self-check-parity` binds the new gate to a self-check verb, so the same task adds one standalone verb under `lyx burler` that takes the review file path, calls `ParseReview` and prints its error on failure, exiting 1.
The gate entry, the verb and the post-gate parse call that one function.
Bound: the entry re-prompts only on a parse failure of the round's own review file, and adds no attempts beyond the gate's existing budget; the verb is read-only, writes nothing and has no mode that skips the parse.
Rationale: the error text already states the fix, and the session that wrote the file is the one that can repair it; the pattern, which wins over the no-new-verbs line, requires the verb beside every mechanical gate.

### D13 — Start verbs never attach a terminal

Decision: `lyx loom start` and `lyx orch start` never attach or switch a tmux client, for any caller, with or without `$TMUX`; attaching to reed is only `lyx reed attach`.
Their attach code paths go.
`--no-attach` stays an accepted, hidden no-op on both verbs, since batten and existing callers pass it.
Every doc, stencil, help text and `doc.go` that describes the start verbs' terminal handoff or `--no-attach` is updated in the same commit, as the repo's CLAUDE.md requires for observable CLI behavior.
`DecideStart`'s attach-only branch becomes a branch that changes nothing and reports the live strand and watcher.
Rationale: operator decision 2026-10-06.

### D14 — Every regression test drives the CLI wiring

Decision: each fix gets a regression test that enters through the cobra command or the CLI-level wiring that reaches it (`lyx shuttle send`, `lyx loom step`/`run`, `lyx orch start`, the orch watcher's tick, `lyx webster recover-batch`/`reset`, the Burler producer row), with fakes only at the tmux, provider and LLM substrate.
Each test sits in the lowest tier its substrate allows, per `PATTERN-test-speed`, extends an existing test where one covers the path, per `PATTERN-test-economy`, and takes its hermetic git and tmux entry points and shared kits from `PATTERN-test-isolation` and `PATTERN-testkit`.
Rationale: the operator's instruction on the board entry.

### Added after Discussion (operator, 2026-10-06): Both test prunes land before implementation

Decision: Webster for this task starts only after test-suite-prune-core and test-suite-prune-rest have both landed on main; the plan writer reads both landed states, since prune-core rewrites tests in `shuttleengine`, `reedengine`, `loomcli`, `shedadapters` and `websterengine` and prune-rest rewrites the orchcli, orchengine, burler, batten and loom tests this task extends.

Rationale: Operator decision 2026-10-06: the two prunes rewrite the tests this task's regression tests build on, so implementing first would conflict with them.

### Added after Discussion (parent, 2026-10-06): Plan now, re-check test targets after the prunes

Decision: Plan-Write writes the plan now against the current tree; the 'reads both landed states' clause of the prune decision applies before Webster, not before Plan-Write. Cards name production code as their main targets and test files by path rather than by test-function glyph where they can. Once both prunes land, the pair is synced with main and the plan's test targets are re-validated, re-planned only where a prune renamed or deleted them.

Rationale: The operator's intent was to use the waiting time for Discussion and Plan, with implementation held until the prunes land; blocking Plan-Write on the prunes defeats that.

## Constraints

- `PATTERN-shuttle-provider-seam`: input-box reading (D1), pacing (D2) and `/reload-plugins` (D10) live under `internal/shuttleengine/claudeengine`; `shuttleengine` and `reedengine` stay provider-free.
- `PATTERN-orch-pane-single-writer`: D10's typing is the orch watcher's own, idle-gated.
- `PATTERN-friction-capture`: D3 still writes the halt note; only the spawned reflection is skipped.
- `PATTERN-refusal-way-forward` and `PATTERN-no-denied-recovery`: D4's failed-removal error (`lyx reed remove`) and D11's envelope (`lyx orch start`) name a way forward that is a lyx verb.
- `PATTERN-gate-self-check-parity`: D12's gate entry, its self-check verb under `lyx burler` and the post-gate parse call `ParseReview`, all in one task.
- `PATTERN-sole-parsers`: D12 parses the review only through `burlerengine`'s existing parser; the reader stays strict.
- `PATTERN-pair-teardown`, `PATTERN-batten-bookend`: D4 and D5 remove strands, never worktrees, and do not touch pair teardown.
- `PATTERN-spawn-observability`: no new process spawns; strand removals in D4 and D5 are logged.
- `PATTERN-test-speed`, `PATTERN-test-economy`, `PATTERN-test-isolation`, `PATTERN-testkit`: D14.
- `PATTERN-comment-line-breaks`, semantic line breaks in every changed `.md`.
- No new verbs, config keys or features (board entry) beyond D12's self-check verb; D11's stopping record is a state field, not config.

## Auto-mode assumptions

- A1: Claude Code's `/reload-plugins` exists in the deployed version and rebuilds the Skill tool's list without ending a turn (D10).
- A2: Claude Code re-injects invoked skills after a compaction, as the board entry states, and `/clear` does not (D9).
- A3: the input box can be told apart from transcript text in a capture, as the existing idle probe already does (D1).
- A4: the shuttle gate's existing attempt budget is enough for one YAML repair (D12); no new budget key is added.
- A5: a fixed settle of a few hundred milliseconds between text and Enter clears Claude Code's paste window on a busy machine (D2); D1 covers the case it does not.
- A6: the cause of the false crash-resume is reproducible through the CLI from the reported history (D6).

## Open risks

- R1: if `/reload-plugins` is absent or does not refresh the Skill tool's list, D10 is a no-op and a new skill stays unknown until a fresh `lyx orch start`; the skill-load step already logs it as `unknown`.
- R2: a provider TUI change to the input box breaks D1's reading; it then fails closed to today's appearance-only check rather than sending blind Enters.
- R3: D8 may skip a reload that a compaction in the middle of a long turn needed, if that turn's end is followed by another before the watcher reads; the role pointer reaches the session on its next cycle.
- R4: if Claude Code does not re-inject skills after a compaction, D9 leaves a session without a skill until its next `/clear` or restart.
- R5: a typed text long enough to collapse into a paste placeholder is never confirmed by D1, so a lost Enter on it is caught only by D2's pacing.

## Acceptance criteria

- `lyx shuttle send` to a pane whose input box still holds the sent text after Enter sends one more Enter and reports success once the box clears; a box that never clears ends in an error naming the pending input after two extra Enters.
- A send to a pane whose input box clears at once sends no extra Enter, and neither does a send whose box holds a draft of other text, including a longer draft that contains a short sent text, a collapsed paste placeholder or a running turn.
- The input box is read only after the settle that follows an Enter, and a test with the sleep injected observes that order.
- `claudeengine`'s text-plus-submit sequences type the text unsubmitted, settle, then send Enter as its own step.
- A `blocked` run halt classified `escalation-to-human` writes `loom-halt.md`, spawns no reflection and reports `friction: skipped`; a `bounce-budget-exhausted` or `producer-hard-failure` halt still reflects.
- After a halt, `lyx loom goto --to <producer>` and a step, the producer's earlier strand is gone before the new one starts; a live attachable run is still attached to, never removed.
- A strand of another producer, or a same-role strand with other outputs, survives a producer re-run.
- A producer re-run whose predecessor's strand removal fails ends the start with an error naming the strand and `lyx reed remove`, and spawns no new strand.
- A failed `recover-batch` leaves no live recovery strand, and `webster reset` removes a live recovery strand of a reset batch.
- A step following a completed `Webster / done` step writes no `loom-crash-resume` note; a step after a driver that died before recording the handoff voucher, mid-producer or right after a finished step, still writes one.
- A crash-resume note names the run-lock reading and the voucher reading, and either lists all history entries or states how many it omitted.
- A watcher started on an old events cursor, whose transcript shows a compaction followed by more than one turn end, types nothing and moves its baseline past that boundary.
- A compaction boundary followed by one turn end that the watcher has not yet read types nothing and leaves the baseline until that turn end is read.
- A compaction boundary with no later turn end types nothing and leaves the baseline.
- A compaction followed by exactly one turn end enters the reload sequence at the pointer step, with `/reload-plugins` typed first and no skills step; a `/clear` reload types `/reload-plugins`, then the skills step, then the pointer.
- After `/reload-plugins` is typed, the next step is typed only on a later tick whose idle probe passed.
- The "The reload sequence" section of `internal/orchengine/doc.go` describes `/reload-plugins` as the first step and no longer says a compaction loses the session's skills.
- `lyx orch start` run right after the watcher received SIGTERM spawns a watcher once the old one has exited, within the bound, instead of reporting it live; when the old watcher outlives the bound, start reports it live and says it is stopping.
- A Burler round whose review frontmatter fails `ParseReview` is re-prompted in the same session with the parse error, and a repaired file lets the round complete with its verdict; a file still invalid when the gate's budget is spent fails the round as today.
- The `lyx burler` review self-check verb exits 0 on a valid review file and 1 with `ParseReview`'s error on an invalid one, and the gate entry returns the same error.
- `lyx loom start` and `lyx orch start` run with `$TMUX` set never call `switch-client` or `attach`, with or without `--no-attach`, and `--no-attach` is accepted.
- No doc, stencil or help text describes a start verb handing over the terminal.
- Every scenario above enters through the CLI wiring named in D14.

## Notes for the plan writer

- Verified send: `sendVerified` and `playInputs` in `internal/shuttleengine/run.go`; `ComposeSend` and `ModelSwitchSequence` in `internal/shuttleengine/claudeengine/startup.go`; the idle probe's input-box reading is the model for D1's capability.
- Halt reflection: `reflectHalt` and `loomPostRun` in `internal/loomcli`; anomaly kinds in `internal/loomengine/anomaly.go`.
- Attach probe: `internal/shuttleengine/attach.go` (`collectAttachCandidates`), used by `internal/shedadapters/singlellm.go` and the Burler adapter's `probeLiveRound`.
- Recovery strands: `internal/websterengine/recoverbatch.go` (`awaitTerminal`, `failBatch` path, `removeStrandIfLive`).
- Crash-resume: `observeEntry`, the handoff voucher and `writeCrashResumeNote` in `internal/loomcli/entryobservation.go`; `loomPreStep`/`loomPostStep` in `internal/loomcli/arm.go`. Candidate causes to check: a history entry appended after `loomPostStep` recorded the voucher, a step path that bypasses `PostStep`, or a second entry observation consuming the voucher.
- Orch: `DecideStart` in `internal/orchengine/launch.go`, `WatcherLive`, the watcher's auto-compaction check and reload steps in `internal/orchengine/watcher.go`, signal handling in `internal/orchcli/watch.go`, attach in `internal/orchcli/handover.go`.
- Loom attach: `internal/loomcli/start.go`; batten passes `--no-attach` in `internal/battencli/wire.go`.
- Burler: `burlerengine.Run` and `repairReportBeforeGate` in `internal/burlerengine/engine.go`; `ParseReview` and `quotedSummaryHint` in `internal/burlerengine/verdict.go`.
