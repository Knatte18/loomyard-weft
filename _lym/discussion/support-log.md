# Support log — creel-hook-delivery-fixes

## Interview

A darn run holds no interview: the writer read the board entry, the code and the Claude Code bundle, and made the calls itself.

## Rejected alternatives

- **Drop the `systemMessage` from the Stop block only.**
  It removes the operator's `Stop says:` duplicate, but the model still gets the mail twice, since both model copies come from the one `reason`.
- **A Stop output with additional context and no `decision: block`.**
  It would keep the command text out of the model's context, but nothing shows such an output keeps the turn going;
  if the turn ended, the mail would be journaled read with no turn acting on it.
- **A shorter Stop hook command, a lym verb or a script in place of the merged shell.**
  It shortens the command the blocking-error copy names, yet both copies would still carry the mail, and it moves the turn-end recording out of the settings the hook tests pin.
- **Fix the HookWake flake in the worker, by draining a buffered wake before it sleeps.**
  The production worker is correct; a coalesced wake only costs one extra probe, so the change belongs in the test's counting.
- **A settle that waits on the fake's pause-read count.**
  A probe already past its pause read when the trigger lands still leaves a buffered wake behind, so the count alone cannot tell the last probe.

## Review rounds

_No rounds yet._

## Question ledger

- Which of the two Stop hooks writes the second copy? Resolved: neither hook writes two; Claude Code renders the one block's `reason` twice (D1).
- Can the hook command text be kept out of the agent's context while the Stop still blocks? Resolved, self-pick: no, so the reason carries a pointer and the mail moves to the additional context (D2).
- Is the note `creel-ring-carries-mail` touched? Resolved: no; the ring and the bell line are unchanged.
- Do the two findings share a cause? Resolved: no.
- What does the HookWake blocked-chain row wait on? Resolved: one probe wait per trigger, which a multi-post trigger breaks (D4).
