```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. For each guard, refusal, branch and test row the plan adds, I checked whether a real entry point can reach it.

**Findings**

1. **BLOCKING — Card 7, sender-notice rule and its test row.** The row "a program sender that never read gets no notice while the journal records the outcome" cannot happen under the card's own rule.
   - Card 7 sends the notice to "a program whose mailbox is known".
   - The fold marks a program's mailbox known on its first post as well as its first read: `internal/creelengine#pool.applyPosted` runs `if isProgramMailbox(rec.From) { p.programs[rec.From] = true }`, and the `pool.programs` comment says "from their first post or read".
   - So a program that posted the marked message always gets the notice. The row fails as written, and D10's acceptance clause ("a program sender that never read gets no notice") is unmet.
   - The only sender that reaches the journal-only branch is EXTERNAL.
   - Fix: track "has read once" apart from `programs`, or restate the rule and the row.

2. **BLOCKING — Card 4 (D1/D3 same-strand) and cards 2/12 (`--parent` from the cwd).** One hold-notice notifier is not on card 4, and its agents resolve `--parent` to a different orch than their hold notice reaches.
   - The missed wiring is `internal/battencli/drive.go:448`, `runner.SetNotifier(func(line string) error { _, err := notifyRunParent(location, c.slug, line) ...`.
   - That runner is `battenCLI.primeShuttle`, which is "built from the prime's own reed geometry". So the tackler and arbiter strands run in the prime worktree.
   - Both stencils render the parent directive (`contracts/stencils/shed/shed-template-tackler.md`, `contracts/stencils/loom/loom-template-arbiter.md`), so card 12 makes them ask with `lym creel post --parent`.
   - From the prime, card 2's cwd-based `orchroute.Asker` resolves to the prime's orch.
   - Their hold notice still goes through `notifyRunParent`, which walks from the run's pair to the nearest live orch: the beam's orch when that orch is live.
   - Result: for a run under a live beam, the question and the hold notice reach different orchs. The criterion "for one held agent … the receiver its `--parent` post resolves and the orch its hold notice reaches are the same strand" fails.
   - `orchroute#TestAsker` does not reach this, because it tests the rule, not the prime-hosted strands.

3. **LOW — Card 8, reply from a former holder of a chased message.** No input can reach this refusal; the plan says so itself (Shared Decision `former-holder-reply-guard`).
   - `internal/creelengine#Store.Forward` refuses a read message (`msg.Status != StatusUnread`), and only read messages are chased.
   - The matching D10 acceptance row is therefore not reachable. The plan records this as a fail-loud guard rather than a tested refusal.

4. **LOW — Card 7, the `isChaseable` condition "is no chase itself".** This state cannot occur.
   - Card 7 posts a chase through `appendPostedLocked` with `Chase`, `Chased` and `ConsumedSeconds` and no `ExpectReply`, so a chase is never marked.
   - The mark condition already excludes it, so the clause is dead.

5. **LOW — Card 8, the same reply case recurs unguarded.** Card 8 redirects a reply to a chase, but a reply to CREEL's other message, the "unanswered" sender notice, gets no such handling.
   - That notice (card 7, `settleUnansweredLocked`) is a CREEL post without `ChaseOf`.
   - `Store.Reply` would address the reply to `&lt;shortname&gt;:CREEL`. Card 7 says the CREEL mailbox is never made known, and `deliverableReceiver` rejects programs.
   - So the reply sits unread forever with no refusal. Card 8 guards this for chases only.

Calls made:
- `grep SetNotifier(` over `internal/` non-test files: needed every notifier wiring to check card 4's coverage; my context held only the three the record names.
- `sed -n 435,460p internal/battencli/drive.go`: needed batten's notifier body, which my context lacked.
- `sed -n 415,434p internal/battencli/drive.go` plus a grep: needed which worktree batten's runner runs its strands in.
- `grep -ln parent_directive` over the tackler and arbiter stencils: needed whether the prime-hosted roles render the parent directive and so use `--parent`.
