# Support log — pause-esc-rewind

## Interview

The session ran autonomously; no operator answered.
Every pick below is an auto-pick, recorded in the Question ledger.

### Turn 1 — exploration

- Read the board entry and the related entries `creel`, `orch-split`, `run-window-bugs` and `reed-multi-window`.
  `creel` will move `lyx shuttle interrupt` behind creel and asks shuttle how keys reach a pane safely, so a guard inside shuttle's interrupt fits it; no other entry plans pause interrupt work.
- Read `pauseengine` (pause, resume, doc), `pausecli/deps.go`, shuttle's session fold and reader, `Runner.Interrupt`, the pane-ready check, `claudeengine`'s idle and startup readers, the orch watcher's hub-pause loop and the `lyx pause` landing commit `ad900dc11`.
- Root cause: `judgeTurnEnd` reads a Stop with outstanding background tasks as `busy`/`background`.
  The pause's `Busy` compares the state name only, so both drivers, idle at their prompts while `lyx shed step` ran in the background, held the grace and got Esc, re-read busy (the background task was still outstanding), got a second Esc, and still read busy.
  Two Escs on an idle prompt open the rewind picker; `claudeengine` already notes that an Escape right after another opens it.
- The board's questions, answered from code:
  pause sends at most two Escs per session, and did send both here;
  resume neither closes a picker nor types into it, because its guarded send refuses a pane with no idle input box and names the refusal "the input box holds a draft";
  `interrupted` means stopped after an Esc and `still_busy` means busy after both, so both drivers landed in `still_busy`;
  Go producers keep running by the shipped design, but a run whose driver session is live is not listed under `go_producers`, so the running shed steps were invisible.
- The orch watcher's hub-pause interrupt repeats the two-Esc loop with an immediate re-probe, so the orch can hit the same picker.
  Corrected in the lens review: its idle probe goes through `judgeReadiness`, which already reads `busy`/`background` as ready, so the orch's only exposure is the stale re-probe after the first Esc.

### Turn 2 — design (auto-picked)

- Busy for the pause means a running turn (D1); background sessions are reported with their task labels and not recorded (D2).
- Every Esc is pane-guarded in `Runner.Interrupt` (D3), and a retry waits a provider settle time and re-reads the pane (D4).
- Report lists redefined (D5), resume's refusal reason corrected (D6), Go producers unchanged (D7).

### Turn 3 — lens review

Forks spawned for premises, goal-scope, board-fit, pattern-fit, scenarios and leanness.
The leanness fork's prompt named the rubric's "attack-surface question" where its lens text names "Relocation and exclusion findings are legitimate", a transcription slip in the spawn; its completeness-before-leanness focus was unchanged.
All six forks returned a report.

- leanness, MEDIUM, D5 vs criteria on a guard-refused second Esc: fixed; a refusal on the retry counts as `interrupted`, criterion 3 names it.
- leanness, LOW, D1 rationale retells the live test: fixed, sentence cut.
- leanness, LOW, D5 closing history: fixed, cut.
- leanness, LOW, D7 (Go producers) repeats Scope: fixed, folded into Scope Out; D7 is now the PATTERN entry.
- leanness + pattern-fit, LOW, `PATTERN-told-geometry` miscited: fixed, dropped; the seam rule now rests on `PATTERN-shuttle-provider-seam`.
- pattern-fit, LOW, `PATTERN-leaf-packages` miscited: fixed, dropped.
- pattern-fit, MEDIUM, guarded interrupt is a cross-cutting rule with no PATTERN entry: fixed, new D7 adds the entry and repoints `PATTERN-pane-text-injection`'s background.
- pattern-fit, MEDIUM, orch watcher settle has no seam: fixed, D4 settles across ticks with an in-memory first-Esc time, the settle told through `Session`.
- pattern-fit, LOW, `ErrNoRunningTurn` gives no way forward: fixed, its message names `lyx shuttle send`.
- pattern-fit, LOW, Turn 3 records the spawn slip: rejected; the lens-review record is the place a spawn deviation is owed.
- premises + scenarios, MEDIUM, orch background premise wrong: fixed in Turn 1, the assumption and the orch criterion.
- premises, MEDIUM, first-Esc-stopped session lands in `already_idle`: fixed with the leanness finding above.
- premises + scenarios, LOW, `Run.Interrupt` has no production caller: fixed, a plan note.
- premises, LOW, resume over a stuck pane waits the send-ready timeout: fixed, added to Open risks.
- scenarios, MEDIUM, orch watcher's handling of `ErrNoRunningTurn`: fixed, treated as idle with no Warn, with a criterion.
- scenarios, MEDIUM, D1 adds a second busy predicate beside `judgeReadiness`: fixed, D1 extracts one shared predicate.
- scenarios, MEDIUM, no scenario for a pre-change entry holding a now-background session: fixed, criterion added.
- scenarios, LOW, guard capture failure unspecified: fixed, fails closed into `refused`.
- scenarios + goal-scope, LOW, D6 fixed text drops the specific reason: fixed, the guard's own reason passes through.
- scenarios, LOW, orch watcher resume during settle: fixed, D4 states the tick's entry-live check.
- goal-scope, MEDIUM, `already_idle` mixes recorded and unrecorded sessions: fixed, new `no_running_turn` list.
- goal-scope, LOW, Goal overstated against D3's bound: fixed, Goal reworded to the pane.
- goal-scope, LOW, `lyx shuttle interrupt` change is a scope addition: fixed, named in Scope In.
- goal-scope, LOW, no by-hand check of the settle assumption: fixed, by-hand criterion added.
- board-fit, MEDIUM, `creel` relies on the sentinel's contract: fixed, D3 states "nothing to stop" is success for stop-only callers.
- board-fit, LOW, overlap with `run-window-bugs` in `Startup`: fixed, plan note.
- board-fit, LOW, orch watcher loop moves under `orch-split`/`creel` later: fixed, plan note.

## Rejected alternatives

- **Drop the second Esc entirely.**
  Simpler, but a turn whose first Esc is swallowed would then run through the pause; the guard and settle make the retry safe, so it stays.
- **Close a picker at resume with one Esc, or detect it from the pane.**
  lyx no longer opens it after D1 to D4, an operator may have opened it deliberately, and the guarded send already refuses to type into it.
- **Retry loop inside `Runner`.**
  It would lose the pause engine's entry-live check between Escs, so a resume landing between them could see its own resumed turn interrupted.
- **A new `InterruptTurn` beside an unguarded `Interrupt`.**
  Two entry points to the same key leave the unguarded one for the next caller to misuse; one guarded `Interrupt` serves every caller.
- **Record background sessions in the entry for a resume line.**
  Their turn was not cut, and they wake themselves when their task ends; a line would start a needless turn.
- **A blocking 2-second sleep inside the orch watcher's tick.**
  It stalls the watcher's other duties and needs wall time in tests; waiting across ticks needs only `Now()`.
- **A busy predicate private to `pausecli`.**
  `judgeReadiness` already holds the same rule inline; two copies drift.
- **Fixed reason text at resume.**
  It still misnames an operator-activity hold; the guard's own reason is already specific.
- **Hold or stop Go producers under a pause.**
  Contradicts the shipped design decision in `ad900dc11` and is not what the board entry asks to change.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): What does the pause treat as busy? — A running turn only (`busy`/`turn`). Background sessions have an idle prompt and Esc does nothing for them.
- Q2 (resolved, auto-pick): Where does the Esc guard live? — In `Runner.Interrupt` and `Run.Interrupt`, so the pause, the orch watcher and `lyx shuttle interrupt` all get it; provider specifics in `claudeengine`.
- Q3 (resolved, auto-pick): Keep the second Esc? — Yes, after a 2-second provider settle and a fresh pane read, at most two per session.
- Q4 (resolved, auto-pick): Should resume close a picker? — No; the guarded send refuses, and the reason text is corrected.
- Q5 (resolved, auto-pick): Are the running shed steps intended? — Yes, per the shipped design; the new `background` list makes them visible.
- Q6 (resolved, auto-pick): Is the orch watcher in scope? — Yes, for its immediate re-probe after the first Esc; it already reads a background orch as idle.
- Q7 (resolved, auto-pick): Does a session claimed but turned away by the guard keep its entry record? — Yes; the claim-before-Esc order stays, it is listed under `no_running_turn`, and the resume line's text covers a session with nothing cut.
- Q8 (resolved, auto-pick): Is the guarded interrupt a PATTERN entry? — Yes, a new entry in the same commit (D7).
- Q9 (resolved, auto-pick): How does the orch watcher wait the settle? — Across ticks, remembering the first Esc's time in memory per entry.
