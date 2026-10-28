```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case and checked the end state against the decision's stated purpose.

**Findings**

1. **MEDIUM — D2 (and D3): standard input is read twice in the merged Stop command.**
   - Case: a post arrives after the last tool call and the turn ends.
   - The merged command runs `hook-read` first, and since D3 `hook-read` reads the hook's standard input to check `agent_id`.
   - `recordingCommand`'s payload step is `cat >> events.jsonl`, which reads the same standard input.
   - Whichever step runs second gets an empty stream. When `hook-read` prints nothing, the turn-end record holds a stamp but no payload, so `ParseEvents` sees no `Stop` line and the run never sees the turn end.
   - The flag gate `[ -f flag ] || exit 0`, copied from `creelMailHookCommand`, would also skip the recording on the common no-mail path.
   - Evidence: `settings.go` `recordingCommand` (`cat >> <events>`); `creelMailHookCommand` exits 0 when no flag stands.
   - Fix: the record should say the payload is read once and handed to both steps, and that the flag test gates only the creel step.

2. **MEDIUM — D2 bound: when Claude overrides the 9th block, the mail it claimed is lost and the run sees no turn end.**
   - Case: mail keeps arriving and the model answers without a tool call eight times in a row.
   - On the 9th `Stop`, `hook-read` claims the mail and prints a block. The mail is journaled read once the output is written, and the merged command skips the record because it printed a block.
   - Claude Code's cap then overrides the block and ends the turn. The claimed mail never reaches the model, and `events.jsonl` gets no record for the real end.
   - The record's bound says "the mail left then is reached by D4 or D5", but the mail claimed in the overridden block is already read.
   - Evidence: hooks docs, Stop input ("after stop hooks have continued the turn eight times in a row, Claude Code overrides the next block"); D2 ignores `stop_hook_active`; `ClaimAtBoundary` reserves and the acknowledgement journals the mail read.
   - Fix: the hook counts its own consecutive blocks, using `stop_hook_active` or a counter, and stops claiming before the cap, or the record states this loss.

3. **MEDIUM — D4 and D2: a stale bell can follow every wake that the Stop block delivers.**
   - Case: an idle strand with a live waiter gets a post.
   - The waiter rings and the model answers the bell with a short sentence. At that turn's `Stop`, the merged recording command (D2) and a new waiter entry (D4) run side by side.
   - The new waiter registers and the daemon checks the ring condition. The mail may not be claimed yet, since `hook-read` is starting in parallel, and the pane may read idle while the Stop hooks run.
   - In that case the daemon rings again: a second bell turn after the mail turn, carrying nothing.
   - D4's bound only names a race with a turn the operator starts. This race is part of the main idle path.
   - Evidence: D4 states that "a newer waiter ends an older one" and that a ring needs claimable mail plus an idle reading; D2 claims in a parallel hook of the same event.
   - Fix: a fresh registration does not ring on its first reading (it waits for a change after it registers or for a settle), or the bound names this case.

4. **LOW — D4: the bell text can lead the model to take its mail through `lym creel read`, where it is collapsed in the pane.**
   - Case: Sonnet gets the bell, which says the mail comes "at your next tool call or when this turn ends", and runs `lym creel read` as its next tool call.
   - `read` claims and acknowledges the mail, so the flag clears before `PostToolUse` runs and no block is printed.
   - The message reaches the operator only as Bash tool output, which the pane folds. That misses D4's purpose of showing every message as a block.
   - Evidence: D6 keeps `lym creel read`; the rewake stderr goes to the model as a system reminder.
   - Fix: the bell says to end the turn and not to run `lym creel read`.

5. **LOW — D5 and D4: a bell can be typed in the window before a waiter connects.**
   - Case: mail arrives just after a `Stop` and before the waiter's `lym` process has connected.
   - The worker sees an idle strand with claimable mail and no live waiter, so it types the bell. The waiter then connects and rings as well, which can give two wakes.
   - "Once per new claimable set" limits D5's typing but not the waiter's ring for the same set.
   - Evidence: D4 counts a waiter live only once its connection is open; D5's condition is "no live waiter".

6. **LOW — D1, D2, D4: how `hook-read` and `hook-wait` learn the event is not stated.**
   - D1 and D2 call the same `lym creel hook-read claude` command for both `PostToolUse` and `Stop`, and the Notes say the capability renders per event.
   - Today the two blocks have the same shape, so nothing breaks. But nothing hands the verb the event: it has neither an argument nor a stated read of `hook_event_name`.
   - Evidence: D1 says "renders per hook event"; D2 reuses the identical command string.

**Each decision, the case used, purpose met or not**

- **D1** (a 5-call turn, a post after call 2): flag test, then `hook-read` reads input, sees no `agent_id`, claims and prints the block; Claude shows the reason and the `systemMessage`. Met.
- **D2** (a post after the last tool call): not met as written (findings 1 and 2).
- **D3** (a foreground fork's Bash call while mail stands): `agent_id` is set, so nothing is claimed and the flag stays; the main thread's `PostToolUse` on the Agent call takes the mail. Met.
- **D4** (an idle strand, post, live waiter): ring, bell turn, Stop block, then the mail turn. Met, with the stale-bell race (finding 3) and the `read` side path (finding 4).
- **D5** (`post --interrupt` with the waiter gone): bell typed, the turn's Stop block delivers. Met; window race noted (finding 5).
- **D6** (agent mail to an idle strand): no notice or reminder typed; delivered through D4 or D5 and D2. Met, but it depends on D2's fix.
- **D7** (`post --interrupt` to a running receiver): interrupt, no `Stop` fires, post stored, flag set, wake through D4 or D5, then the Stop block. Met.
- **D8** (`MessageLine` for `ly:orch`): `[creel] #1 from "ly:orch": …`; the claim budget counts `MessageLine`, so the 2 extra bytes are budgeted. Met.
- **D9** (launch line with a deny notice): the sentence is appended after the deny notice through `sh.Quote`; the resume line is built the same way. Met.
- **D10** (`lym creel post --help`): carries the rule. Met.
- **D11** (orch stencil): the reminder text is replaced. Met.

**Tool calls**

- I read `recordingCommand` in `settings.go` and `ClaimAtBoundary` in `verbs.go`. My context did not hold how the recording reads standard input, or when a claim reserves mail, which findings 1 and 2 need.
