```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged the reviewer's proposals that the fixer carried out (F1, F2, F3, F5, F6, F7, and F4 in its changed form).

**Findings**

1. MEDIUM. The F5 fix contradicts the F6 fix.
   - Changed text: D3, "Every way the ring's answer path ends without a confirmed acknowledgement releases the claim".
   - Summary: Under that rule, the F6 window would release the claim, but the F6 Bound says the same window leaves the mail journaled read.
   - Case: The daemon receives the ack and runs `reading.Ack()`, which journals the read. Its confirmation write is then lost, so `awaitAckConfirmation` returns `connLostError`. From the client's side this is an end "without a confirmed acknowledgement", so the D3 rule says release. D3's second Bound and the Constraints exception say the mail is read and lost. D5's `released` sentence covers the same "every early end", so it would log `released` where the Bound says `acknowledged` records it.
   - Evidence: `server.go` `handOut` calls `reading.Ack()` before `d.write(... OK)`; D3's new sentence; D3 Bound 2; D5's `released` sentence.
   - Fix: limit the release rule to ends that come before the daemon journals the acknowledgement.

2. MEDIUM. The F1 proposal overstates what the scrub does.
   - Changed text: D1's last rationale bullet, "reed's Claude launch scrubs every `CLAUDE_CODE_`-prefixed variable (… applied in `internal/reedengine/lifecycle.go`)".
   - Summary: The scrub runs once, on the tmux server's environment at boot. The Claude pane command then sets `CLAUDE_CODE_` variables of its own. So "the launch scrubs every prefixed variable" is not what the code does.
   - Evidence:
     - `lifecycle.go:451` applies `e.scrub.Apply(os.Environ())` to the `tmux` server spawn's `cmd.Env`.
     - `claudeengine/command.go:106-112` adds `CLAUDE_CODE_FORK_SUBAGENT` and `CLAUDE_CODE_PROMPT_CACHE_TTL` on the pane command, because "the reed server env is scrubbed of CLAUDE_CODE_* at boot".
   - The conclusion still holds for `CLAUDE_CODE_SIMPLE`, since lym never sets it. The wording should say: the tmux server env is scrubbed at boot, and lym's pane command sets no `CLAUDE_CODE_SIMPLE`.
   - The bullet also leaves out a path lym does not control: an `env` key in the operator's Claude settings.

3. MEDIUM. The F4 criterion, as changed, cannot fail on the gap it is meant to catch.
   - Changed text: the D3 real-session criterion, "the operator sees the mail at most once, beside one `creel mail` row".
   - Summary: "At most once" passes when the terminal draws no body at all. Yet the new Open risk says this criterion "shows which", and the Goal asks for "the operator once".
   - Case: The terminal draws only the `creel mail` row, so the operator sees zero copies of the mail. The criterion passes, the Open risk counts as settled, and the Goal's operator-once is unmet on the ring path with nothing flagging it.
   - The review's proposal said "sees the mail once".
   - Fix: make the criterion record which outcome holds, or require exactly once.

4. LOW. The F2 bound states more than the evidence shows.
   - Changed text: D1 bullet 3, "a claim of more than 5 lines shows the operator its first 5 there", and the D1 criterion, "shows the operator its first 5 lines in the stop-hook summary row".
   - Summary: The clamp `Is([...], {maxLines:eu})` runs over the whole joined list. That list holds every Stop hook's lines, with `Stop hook error:` lines ahead of the `Stop hook feedback:` lines. The clamp's own output, such as whether it spends a line on a "+N lines" marker, was not read.
   - Case: Another Stop hook in the merged settings reports an error. The creel claim's lines then start below it, so fewer than 5 of them show. The criterion as written would fail on a correct implementation.
   - Evidence: the `stop_hook_summary` source quoted in context: `Is([...h&amp;&amp;b?[b]:[],...g.map(...Stop hook error...),...f.map(...Stop hook feedback...)].join("\n"),{maxLines:eu})`.

The F3, F6 and F7 proposals hold. I found no case where they are wrong beyond finding 1, which F6 shares with F5.

Tool calls:
- I read `internal/reedengine/lifecycle.go:440-480` and grepped `internal` for `CLAUDE_CODE_`. My context showed only the scrub's prefix list, not where the scrub is applied or whether lym sets `CLAUDE_CODE_` variables after it.
