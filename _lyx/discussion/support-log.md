# Support log — shuttle-send

## Interview

The session ran autonomously, so no operator answered; each turn below is a question the design had to settle, answered from the board, the code and the incident chain.

1. What failed in #442/#443?
   The skill-load send in shuttle's start typed into a freshly started Burler half straight after the startup probe passed; `confirmSubmitted` gave up after 2 extra Enters (about one second), and the error carried no shuttle classification, so the shed boundary marked nothing transient and the run halted `failed`.
2. What failed in #444?
   Burler's `stopHalf` removed the other half's strand through a raw `RemoveStrand`, leaving its `run.json` reading `running`.
   The re-step's probe found an untracked strand behind a young running record, which `leftoverThenAgeVerdict` refuses as a possible live agent.
   The refusal is right for an unknown run and wrong for one lyx itself stopped.
3. Where should readiness come from?
   `shuttle-state` landed hook signals in shadow mode, including `UserPromptSubmit` as `turn_start`.
   Picked: the pane's idle reading plus the raw `turn_start` signal, used only to delay or confirm, never to end a run (D1, D3).
4. What should a lost submission become?
   Picked: at start, a not-started run through `abandonStartup`, reusing `ErrNotStarted` and the agent-start transient class (D4); mid-run, a gate re-prompt that stays at the boundary (D6).
5. How does attach tell a stopped half from an unknown one?
   Picked: shuttle owns one stop verb that writes a record-only terminal outcome `stopped` before removing the strand; burler, webster and attach use it (D7), pinned by a new PATTERN entry (D8).
6. What about the replayed-boundary send from `parent-review-hardening`?
   Covered by D1's signal condition applied at the gated boundary (D6).
7. Testing approach: untagged tier with the injected sleep and clock seams and the existing shuttle fakes, extending the `confirmSubmitted` table test; no llm-tier reproduction.

## Rejected alternatives

- **Raise `sendExtraEnters`.** A larger count is still a fixed budget unrelated to how loaded the machine is; a time window with backoff (D3) adapts.
- **Mark `ErrSubmissionNotLanded` with a new transient class.** PATTERN-transient-stop treats a new class or site as a change to what drivers re-step unattended; the start-time case already fits `TransientAgentStart` once its definition names a session that never took its first input.
- **Make attach's "cannot be confirmed" outcome transient (#444's alternative).** It would let a driver re-step beside a genuinely unknown live agent once the window passes; the refusal stays for unknown runs and lyx-stopped runs stop reaching it.
- **Reuse `died` as the stop record.** It hides that lyx stopped the run on purpose, which is what a diagnosis of a halted run needs to see.
- **Burler writes the run record itself.** Shuttle owns the run directory; a second writer is how the bug arose.
- **Gate sends on the session-state fold's derived state.** `shuttle-state` sequences fold consumers as separate tasks; the raw `turn_start` fact is enough here and carries no derivation risk.
- **Wait for a `SessionStart` hook before the first send.** Not recorded today, and nothing shows it implies input readiness; the idle reading covers the case.
- **Leave webster's reclaim on its own remover.** It never hit the attach refusal, but two stop paths writing different records is the shape that produced #444.
- **Stop writes `done` over a run whose output files exist.** A `done` written outside finalize skips the gate's final evaluation and the run's cleanup; keeping `running` lets attach's existing file-contract-first branch harvest it through a real `Wait`.
- **Let a missing turn end hold a send for the whole idle window.** A crashed hook or an interrupted turn would then fail every send as busy against an idle pane; the interrupt reading and a 10 s idle-pane override bound it.
- **D1 accepts a box holding the previous failed text and the next send replaces it.** It needs a record of the failed text across sends and processes, and a `lyx shuttle send` from another process has none; clearing at the failure keeps D1's empty-box reading the one rule.
- **Retry an unsent re-prompt only on the writer's next turn end.** The writer never received the re-prompt and stays idle, so no turn end arrives and the run idles to its deadline.

## Review rounds

- Round 1 (BLOCKING, ten findings, all fixed): D6's re-prompt retry gained a per-tick trigger and a deadline cap on in-loop sends; D7's stop verb consults the file contract first, marks the handle so its own `Wait` keeps `stopped`, and aborts on a failed record write; D1 bounds an unmatched `turn_start`; D4 re-reads before teardown and names its residual; D2's settle runs inside D3's window, which now opens at typing; acceptance criteria gained the missing behaviours; Open risks names Q8; the defaults bullet and incident chain were dropped as restatements.
- Round 2 (BLOCKING, eight findings, all fixed): D3's failed send clears its own text so D6's retry finds an empty box, and D4's late-landing check moved onto the pre-clear reading and the signals; D6 restores a cleared boundary under D1's override and takes one deadline budget per send, with the appearance check inside D3's window and a guaranteed post-Enter read; burler stops a half through the handle form and `Run` gains a mutex for the mark; the completion-signal tripwire gains `runOutcomeStopped` as a marker; D1 names `SessionProber` and the turn-end kinds; D8 is scoped to code outside shuttle.

## Question ledger

- Q1 (resolved, auto-pick): idle wait window — 60 s, new key `send_ready_timeout_s`. A loaded machine's startup redraw fits; a genuinely busy agent fails visibly.
- Q2 (resolved, auto-pick): submit window — 30 s, new key `submit_confirm_timeout_s`, backoff from `submit_redraw_settle_ms` doubling to a 5 s cap.
- Q3 (resolved, auto-pick): lines of pane in a send error — 15 non-blank lines.
- Q4 (resolved, auto-pick): stop record value — new record-only `stopped`, never a `Result.Outcome`.
- Q5 (resolved, auto-pick): include webster's reclaim in the stop verb — yes, one stop path.
- Q6 (resolved, auto-pick): `Runner.Send` (the `lyx shuttle send` CLI) also waits for idle — yes; typing into a running turn is the behaviour this task removes.
- Q7 (resolved, auto-pick): a scan test for `PATTERN-shuttle-stop` — only if cheap; review-enforced otherwise.
- Q8 (open, for the operator at review): whether the widened `TransientAgentStart` definition is acceptable, or a session that never took its first input deserves its own class.
- Q9 (resolved, auto-pick, review round 1): the idle-pane stretch that overrides an unmatched `turn_start` — a fixed 10 s, no config key; a running turn shows its hint on every poll, so 10 s of idle pane is strong evidence.
- Q10 (resolved, auto-pick, review round 1): stopping a run whose output files all exist — keep the record `running` and remove only the strand, so attach harvests it.
- Q11 (resolved, auto-pick, review round 2): what a not-landed send does with its text — clears it through an engine-owned choreography, only when the box holds this send's text, a prefix of it or a paste placeholder, never with a key that interrupts a running turn.
- No tool failures were observed during this session.
