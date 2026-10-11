# Support log — burler-multi-llm

## Interview

The session ran autonomously: no operator answered, so every turn below is a question the writer posed and settled from the board, the code and recent commits.

1. **Which layer moves onto seats?**
   The board title says "on `MultiLLMProducer`", but `BurlerProducer` carries round resolution, hydration, retry and the Bouncer hand-off, and `MultiLLMProducer` maps a chair result generically.
   Settled: `burlerengine` drives `seatengine` underneath the unchanged `BurlerProducer` row (D1).
2. **What does the seat table look like?**
   Board: fixer is `chair`, reviewer `advisor-1`.
   Settled: prefix `burler`, the existing orchestrators become the seat stencils, gates and outputs as today (D2).
3. **How does review-before-fix survive, given seats treats an advisor's death as advisory and gates only the chair?**
   Settled: a required advisor with an acceptance hook, plus a per-advisor gate; the hook is today's `reviewReady` (D3).
4. **Do Burler's failure rules change?**
   No: `decideRound` reads a richer `SeatResult` (D4).
5. **What happens to `fix_start: after-review`?**
   Seats starts the chair first; the hub runs `parallel`. Settled: retire the key (D5).
6. **Fan lenses: advisor seats or forks?**
   The board leaves it to Discussion. The Discussion bench and the twill commit show fresh-session advisors re-read code at full price. Settled: forks of the reviewer seat (D6).
7. **Warm cache.**
   `lyx shuttle await` landed in 50cf6a425 with a 250 s default cap. Settled: the chair runs it in its turn exactly as the fixer runs `await-review` today (D7).
8. **Does the seat channel get used inside a round?**
   Settled: only after the marker, only to ask about a finding; the marker stands in for an up line that may never land on a chair waiting in its turn (D8).
9. **Reviewer stall visibility.**
   Seats advisors are `QuietHold`; today the reviewer notifies the parent. Settled: a required advisor keeps its notice (D9).
10. **No false stall notices.**
    lyx cannot see the Claude Code inbox, so "unanswered batch" is read as an advisor mid-turn by shuttle's session-state fold, plus up lines not yet landed; a settle interval covers SendMessage delivery; the notice is deferred, never dropped (D10).
11. **Resume.**
    Settled: today's both-live rule on the seats probe (D11).

## Rejected alternatives

- **Put the Burler row on `MultiLLMProducer` directly.** Every Burler rule (round resolution, hydration, retry, verdict parse, audit, Stuck hand-off) would move into a generic producer or a wrapper around it; nothing is gained over driving `seatengine` from `burlerengine`.
- **Lens advisors as seats.** Each lens a fresh session on its own model: breadth by model, but each re-reads the code at full price, the fan's consolidation and exact-N audit would need rebuilding, and a round would hold up to 17 panes.
- **Keep `after-review` by deferring the chair's start in the seats engine.** Contradicts the chair-first rule just landed, for a knob no hub uses.
- **Keep `lyx burler await-review` as an alias.** The board says it goes; the general verb covers it, and the deploy-window risk is bounded and recorded.
- **Let the advisor's ordinary done end release the fixer, with the chair reading the file itself.** Drops Go's acceptance of the review before the fix, which PATTERN-review-round requires.
- **Suppress every chair notice while any advisor seat is live.** In Burler the reviewer's pane stays up through the fix, so a stalled fixer would never notify.
- **Suppress at the turn end only, with no deferral.** A chair that ends its turn the instant after sending a batch sees an advisor not yet started and notifies falsely; a chair suppressed at its turn end whose advisor later goes idle would never notify.
- **Track batches in a ledger the chair writes.** Adds stencil-enforced bookkeeping Go must trust; session state is already recorded.
- **Include the shared seat directives unchanged, or include them and name which of their lines Burler overrides.** Their wording opens the channel before acceptance, lets the chair ask for a wrap-up and has the chair end its turn for an up line; carrying both rule sets in one prompt leaves the agent to pick a winner.
- **Forbid all chair–reviewer messages.** Safe, but discards the one capability the seat table adds; scoping it after acceptance keeps independence.
- **Make the settle interval a config key.** No one asked to tune it.
- **Bound the deferral by the advisor's own deadline.** Every seat shares the table timeout and the chair starts first, so the chair's deadline always comes first and the bound never fires; a wedged mid-turn advisor would hide a chair stall for the chair's whole run.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Which layer runs on seats? — resolved, auto-pick: `burlerengine` over `seatengine`, `BurlerProducer` unchanged (D1). Rationale: only the agent wiring is replaced; Burler's rules stay where they are.
- Q2 Strand names? — resolved, auto-pick: `burler-chair`, `burler-advisor-1` (D2). Rationale: the board's seat names; `SeatRole` forms them.
- Q3 How is the review accepted before the fix? — resolved, auto-pick: required advisor plus `Accept` hook (D3). Rationale: PATTERN-review-round.
- Q4 `fix_start`? — resolved, auto-pick: retired (D5). Rationale: hub runs `parallel`; keeping it fights chair-first.
- Q5 Fan lenses? — resolved, auto-pick: forks of the reviewer (D6). Rationale: bench evidence on advisor re-reads; audit untouched.
- Q6 Chair–reviewer channel? — resolved, auto-pick: after the marker, questions about findings only (D8). Rationale: independence kept, capability used.
- Q7 Reviewer hold notice? — resolved, auto-pick: required advisor not quiet (D9). Rationale: keeps today's stall visibility.
- Q8 Stall-notice rule? — resolved, auto-pick: deferral on mid-turn advisors and pending up lines with a settle interval (D10). Rationale: Go-observable, never drops a real stall.
- Q9 Resume? — resolved, auto-pick: both-live only (D11). Rationale: matches today.
- Q10 Does a SendMessage-started advisor turn fire the turn-start hook? — open; recorded under Open risks for a live check.
- Q11 In-flight fixers across the deploy? — open; recorded under Open risks, mitigated by deploying with no Burler round in flight.
