```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I took every pair of decisions that act on shared state and checked each acceptance criterion and constraint against the decisions it touches.

**Pairs checked**
- D1 × D2 (`--parent` route vs widened orch trust): held. D2 trusts any orch that D1 can resolve.
- D1 × D3 (`--parent` receiver vs hold-notice receiver): broken for an agent running in a beam (F1).
- D1 × D4 (LLM driver `--parent` vs batten's run notices): held. `lym loom start` runs the driver in the pair, so both walks start with `NotifyParent` from the pair.
- D1 × D10 (marked `--parent` question whose resolved orch dies): held. The holder-gone rule makes it `unanswered` at once.
- D2 × D6 (directive trust vs appended system-prompt trust): held. Both name parent, orch and program mail.
- D2/D6 × D10 (CREEL-signed chase acted on as program mail): held. The chase body is fixed and carries no sender text.
- D3/D5 × D9/D10 (orch answers a hold notice by plain post; the agent's question carries `--expect-reply`): broken (F2).
- D3 × D8 (hold notice keeps `MessageTail`; `lym shuttle send`, the only thing that appended it, is removed): held. The orch types the tail itself, and D8 re-points the docs.
- D5 × D10 (every question carries the mark; D5's rationale names the chasing decision): broken cross-reference (F4).
- D5 (chair asks advisor with `--expect-reply`) × D10 chase: held.
- D8 × D5 (orch must not put a question in an inject; inject replaces `shuttle send`): held.
- D8 × D10 `post --interrupt` (needs a `CreelHooks` receiver): held. D8 also lists `creel interrupt` and `creel inject` for pane-mail strands.
- D9 × D10 (mark follows the holder; chase only after a read): held. A read message cannot be forwarded, so the holder is fixed once a chase can run.
- D9 × D10 (reply to a chase from someone other than the chased message's holder): broken (F3).
- D10 frozen-clock rule × D10 restart derivation (same chase-clock state): broken (F5).
- D10 × D11 (`chased` and `unanswered` events): held.
- D10 × D12 (report reads chases and outcomes from the log over the kept window): held at the level of decision interaction.
- D7 × D5 (seat requirement removed; chair–advisor moves to creel): held. The requirement is only added in Go (`internal/seatengine/prompt.go:166`); no yaml names `peer-messaging`.
- D13 × D5 (scan vs moved sites; webster fork line allowlisted): held.
- Constraint creelcli/orchcli cycle × D1: held. `go list -deps ./internal/creelcli` already includes `orchengine`, and `hubgeom` is imported directly.
- Constraint daemon-wakeups × D10 timer: held as stated.
- Acceptance D1, D3 "in a beam the same strand": contradicted by D3 (F1).

**Findings**

**F1 — MEDIUM — design — D3 vs D1 and acceptance "D1, D3".**
D3 keeps every hold-notice wiring as it is and claims they use "the rule D1 resolves with". They do not for an agent in a beam:
- `burlercli/wiring.go:153`, `loomcli/wiring.go:465` and `webstercli/wiring.go:212` call `orchcli.NotifyParent(location)` whatever the worktree kind.
- From a beam, `parentOrchStrand` (`orchcli/notify.go`) starts the walk at the beam's recorded parent and skips the beam's own orch.
- D1 resolves from a beam with `NotifyFrom`, which starts at the beam itself.

So for a held agent in a beam, the acceptance criterion "D1, D3 … in a beam … are the same strand" fails unless D3 changes the wiring, and D3 says the notice "keeps going through the runner's notifier".

**F2 — MEDIUM — design — D3/D5 hold-notice answer vs D9/D10 reply rule.**
- D2 tells a held agent to ask with `--parent --expect-reply` and end its turn, so the orch gets both the marked question and the SHUTTLE hold notice.
- D3/D5 tell the orch to answer the hold notice "by posting to the strand it names". That is a plain post, not `--reply`.
- D9 counts only a `post --reply` by the holder as a reply.

An orch that follows the hold-notice rule answers the agent, yet creel chases the orch twice and then sends the agent an `unanswered` notice signed CREEL, which D2/D6 tell it to act on. The `lym report replies` counts are inflated too. The record does not say which answer route the orch uses when both arrive, nor that a hold-notice answer should be a `--reply` to the marked question.

**F3 — MEDIUM — design — D10 "from anyone else it is an ordinary reply to the chase".**
- Only a message's holder may reply (creel `doc.go`, "Mail"). So a non-holder can reply to a chase only after the chase was forwarded to it.
- That reply goes to the chase's sender `&lt;shortname&gt;:CREEL`. CREEL is a program that never reads, and a receiver must be "a strand name or the name of a program that has read once" (`doc.go`, "Receivers").

The record leaves the outcome undefined (refused or journaled undelivered), and the acceptance row "by another strand only the chase" asserts a delivery that has no receiver. The new routing edge has no stated bound.

**F4 — LOW — consistency — D5 Rationale.**
"every question carries the mark D8 chases": D8 removes the shuttle verbs. The chase is D10.

**F5 — MEDIUM — design — D10, frozen clock vs restart derivation.**
D10 freezes the chase clock while the holder is dormant, paused or held by another sender. It also says the daemon "derives the chase state from the journal (read time, `chased` records)". Read time alone cannot rebuild a clock that stopped and started, so after a restart a chase can fire early.

The inject TTL that D10 cites persists the time it has used in `aged` records (`ConsumedSeconds`, `agedPersistInterval`; `creelengine/deliver.go:22`, `:603-620`). Its clock also does not stop for a hold: `ageInjects` (`deliver.go:555-575`) counts whenever the receiver is live and running. So "the rule creel already applies to an inject's TTL" is not the rule D10 states, and the journal record D10 needs is not named.

Tool calls:
1. Grepped `SetNotifier`/`NotifyParent`/`NotifyFrom` wiring across `internal`, to check which walk each hold-notice notifier uses for a beam location. My context held only four wirings, without the drivewiring and landingdeps sites.
