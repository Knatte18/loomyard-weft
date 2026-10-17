# Support log — creel-notices

## Interview

The session ran autonomously with no advisors, so every turn below is the writer's own reading and pick.

### Turn 1 — scope from the board

The board entry fixes the stage: notices and the watcher's injects through creel, the watcher reduced to cycle logic, the note request as mail answered by `written <path>` or `defer`, and an advance only on that reply or on creel's confirmation of a command inject.
Folded in: content-free reminders (operator decision) and the notice-log removal (orch recommendation).
Every caller of `orchengine.QueueNotice` and `QueueConditionalNotice` was listed, since the queue goes: batten, `orchcli.NotifyPrime` (burler, webster, loom driver hand-back, loom landing, hibernate, shuttle's hold and dialog notices), tender's revive and the watcher's pause-resume line.

### Turn 2 — how a command reaches the pane

The watcher's `/clear`, `/compact`, plugins, color and skills steps are shuttle session operations that send provider key sequences, while creel's inject types one text line.
Picked: a command inject naming one of a closed set of operations, run through a new `Delivery` operation (D3), restricted to program senders.

### Turn 3 — the note gate and the hold

The reply carries the path, so the gate checks the path and the file; the turn-end gates fold into creel's idle check (D4).
A plain `lyx creel read` returns the oldest unread message, which during a cycle may be a batten notice, so the reminder under a hold names `--from <owner>` (D2).

### Turn 4 — sender identity

A program claim from a non-orch pane is refused by creel, and some notifiers run inside driver or webster panes.
Picked: claim the strand when `$LYX_STRAND_NAME` names a non-orch strand, else the module's program name (D6), leaving identity unchanged.
The current rule: the caller claims the strand named by the environment, and creel's refusal for running in no strand pane triggers one retry as the program (D6).

### Turn 5 — what keeps a direct pane action

Creel refuses an interrupt under a covering pause, which is when the hub-pause interrupt is sent; the watcher keeps it (D9).

### Turn 6 — lens review

All six forks returned.

- scenarios, BLOCKING, persisting an id before creel has issued it: fixed; D5 now persists the step as pending with no id and looks it up in creel's journal after a restart.
- scenarios, notices lost when the orch strand is removed: fixed; D6 states the bound and D8's fallback reads the undelivered mail in `lyx creel log`.
- scenarios and goal-scope, D1 removed a guard with no bound: fixed; D1 states the bound, and holds and command injects on an orch strand are program-only.
- scenarios, the hold across a pause, a restart and a rebind: fixed in D5, with acceptance criteria.
- scenarios and premises, tender keeps no memory for "one pending revive": fixed; D7 uses a journal lookup.
- scenarios, reply validation (trimming, sender, stale id): fixed in D4.
- scenarios, D3 command injects reach any strand: fixed; restricted to orch strands.
- scenarios, acceptance gaps for sender identity: fixed. The gap for a dormant orch was rejected, since D3 now states that the dormant queue keeps a command inject.
- goal-scope and premises, D9 keeps the idle probe under the board's "probe goes": fixed; named in Scope Out and D9 as a bounded departure.
- goal-scope, D5 advances on `unconfirmed`: fixed; stated as a bounded departure, with an acceptance criterion pinning shuttle's outcome per command.
- goal-scope and pattern-fit, the operator's PATTERN text was paraphrased: fixed; the shortened sentence is quoted in full and kept as an open risk for parent review.
- goal-scope, D2's `--from` read as an unlicensed addition: fixed; D2 says it follows from the hold.
- goal-scope, observable CLI changes missing from Scope: fixed.
- goal-scope, the orch wake-up rule was unstated: fixed in D6.
- goal-scope, the live hold has no bound: fixed in D5.
- pattern-fit, program names: fixed; formed through `FormatProgram`, owned as module constants, and the naming hedge dropped.
- pattern-fit, the `/rename` exemption: fixed in the PATTERN text.
- pattern-fit, the orchengine import allowlist: fixed in Constraints.
- pattern-fit, the slash-spelling clause: fixed in D3.
- premises, the watcher inheriting `$LYX_STRAND_NAME`: fixed; D10 says the watcher always signs as its program.
- premises, confirmations (creel's idle reading is the same probe, holds filter as assumed, the batten window is no strand): no change needed.
- board-fit, overlap with stage 3's notice wording and stop-report dedup: fixed in Scope Out.
- board-fit, naming the note request as stage 3's first expected-reply mail: rejected; reply chasing is stage 3's design.
- board-fit, `orch-split`, `codex-engine`, `pause-background-work`: informational, no change.
- leanness, repeated auto-mode bullets: fixed. Contradiction in Constraints: fixed. The Notes reasoning and the test-tier clause: cut. Open-risk justification: cut.
- leanness, trimming D4's because-clause: kept as one clause, since the plan writer needs it to trust that creel's idle gate replaces the turn-end gate.

## Rejected alternatives

- Typing the note request as an inject asking for a reply: the board fixes the request as mail.
- One-line text injects for `/clear` and `/compact` built in orchengine: puts provider spellings outside shuttle and skips shuttle's session-start expectation for a clear.
- Conditioned injects in creel for tender's revive: creel would learn session states, which belong to shuttle; the TTL bound in D7 covers the race.
- Downgrading a refused program claim to the pane's strand: changes a creel identity guard to serve one caller.
- Keeping the notice log as a second copy: the journal already holds every notice.
- Routing the hub-pause interrupt through creel: needs a pause bypass in creel's interrupt guard.
- Rewording `PATTERN-orch-pane-single-writer` to every agent pane now: false until stage 3 moves shuttle send behind creel.
- Letting any privileged sender hold the orch and running a cycle without the hold when refused: under a foreign hold, creel holds the watcher's own mail and injects too, so the cycle would stall anyway.
- A client-chosen idempotency key on posts and injects: adds a protocol field where a journal lookup suffices.
- Re-delivering leftover `notices/` files at the deploy: one-off migration code for at most fifty informational lines the board check already covers.

## Review rounds

### Round 1

Every finding was fixed; the fixer-report holds the dispositions.

- F2: the relaunch path needed a creel verb to drop a program's own queued injects; picked `withdraw` over binding injects to the strand GUID, since creel addresses strands by name and a GUID binding would change every inject.
- F4: the Go client already runs a read handler before it acknowledges, so the watcher persists the gate's outcome there; creel's `log` carries no body and no `reply_to`, which rules out the journal lookup.
- F8: a sender-resolution refusal holds the cycle and shows as `stuck`, since a cycle without the hold was already rejected.
- F12: the pause-resume line goes as mail, which has no TTL, instead of a tracked and requeued inject.
- F14: the orch revive's lookup and inject move into a `pauserecord.Update`, the lock tender's task-session revive sends already take, so no new lock is added.

### Round 3

Every finding was fixed; the fixer-report holds the dispositions.

- F1: a failed first withdraw now makes `lyx orch start` start no session, over accepting an old `/clear` in a continued or adopted session; the exposure left after a failed second withdraw is accepted and stated.
- F2, F3: the Stop times come from `ParseSessionSignals`, which already pairs stamps and holds back a trailing stamp run, set by `Runner.ReadEvents`, over a second pairing in the claude `ParseEvents`.
- F4: a typed refusal code on creel's response, so `NotifyPrime` never matches message text.

## Question ledger

- Q1 Command delivery shape — auto-pick: closed command set through the `Delivery` seam (D3).
- Q2 Reminder under a hold — auto-pick: names `lyx creel read --from <owner>` (D2).
- Q3 Stencil line telling agents a reminder may wait (board open point) — auto-pick: none, since when an agent reads its mail is part of stage 3's reply-discipline rules, which Scope Out leaves to `creel-agents`.
- Q4 Keep the journal `subject` (board open point) — auto-pick: keep; `lyx creel log` uses it.
- Q5 Remove the notice log (orch recommendation) — auto-pick: remove (D8).
- Q6 Operator PATTERN text naming the notice log — auto-pick: drop "and the notice log", since D8 removes it in the same commit.
- Q7 Notice sender identity — auto-pick: strand claim when the environment names a non-orch strand, one program retry on creel's no-strand-pane refusal, else program (D6).
- Q8 Tender's conditional notice — auto-pick: tender-side check, TTL at most one minute, one pending at a time found through a journal lookup (D7).
- Q9 `unconfirmed` outcome of a command inject — auto-pick: advance for `/clear`, the color and plugins steps, the bind color and steps that wait on their own signal (D5); recorded as an open risk.
- Q10 Creel outage in the watcher — auto-pick: held and logged, not counted toward the tick-error cap (D5).
- Q11 Hub-pause interrupt — auto-pick: stays in the watcher (D9).
- Q12 Leftover queued notices at the deploy — auto-pick: not delivered; cleanup removes them.
- Q13 Watcher program name — auto-pick: `ORCHWATCH`, which the program grammar accepts.
- Q14 Who may hold or command-inject an orch strand — auto-pick: programs only (D1, D3), so no foreign hold can block a cycle.
- Q15 Restart between creel's answer and the id save — auto-pick: persist the pending step with its persist time, then look up a matching inject since that time in the journal (D5).
- Q16 Old cycle's hold and injects at a relaunch — auto-pick: `lyx orch start` withdraws ORCHWATCH's queued injects and releases its hold through a new `withdraw` verb, and starts no session when the first withdraw fails (D5).
- Q17 Pause-resume line's carrier — auto-pick: mail, which cannot expire (D9).
