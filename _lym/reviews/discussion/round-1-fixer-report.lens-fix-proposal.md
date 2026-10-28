```yaml
lens: fix-proposal
outcome: reported
```

Proposals checked: F1 (cap at `post`), F3 (count starts at the cap after a restart), F4 (new running-turn reading, guarded `Idle` for the bell), F5 (`CreelHooks` mark), F6 (`hook-read` writes straight to the hook's output), F7 (no stamp on a blocked Stop), F8 (refuse `--interrupt` to an unmarked receiver), F9 (D6 rationale) and F11 (Windows research). F10 was fixed differently.

**MEDIUM: F1's line cap breaks a program poster's notice.**
- **Changed text:** D1, "`post`, `--reply` included, refuses a message whose `MessageLine`… would pass the 8 KiB claim budget, with the refusal's existing way forward: write the text to a file and mail its path."
- **The case:**
  - `internal/loomcli/decision.go:278-281` builds `decisionNoticeLine` with each entry's whole decision and rationale inline, with no size bound. It does this for a one-entry notice and for every notice with `inline` set.
  - A decision plus rationale over about 8 KiB used to be accepted, up to 64 KiB.
  - Under the line cap, LOOM's post of that notice is refused. LOOM is a program and cannot take the "write it to a file" way forward, so the decision notice never reaches the run's agent.
  - That notice is the channel the shuttle prompt tells roles to act on.
- **What the record misses:** D1's Bound names only senders who can follow the file way forward. The record names no program poster that builds an unbounded body.
- **What is needed:** the record should state how LOOM's notice stays under the cap, for example by falling back to headings only, as a several-entry notice already does when `inline` is unset.

**LOW: F1's cap also applies to replies to EXTERNAL, which never pass through a hook.**
- **Changed text:** D1, "`--reply` included".
- **The case:** `Store.Reply` (`internal/creelengine/verbs.go:92-94`) shares `validateBody`. D10 and D9 say a reply to EXTERNAL reaches only a waiting `post --wait-reply`, not a hook field. The new cap still refuses an agent's reply over 8 KiB to an operator's `--wait-reply`, which no field cap needs.

**LOW: F6's direct write can lose the turn-end record when the hook is killed.**
- **Changed text:** D2, "On its exit 0 the command appends nothing…" and the Bound "A hook killed between its write and the acknowledgement leaves the claim unacknowledged, so the mail comes again… never lost."
- **The case:**
  - If Claude Code kills the merged command at its timeout after `hook-read` wrote the block, the shell never reaches its record step. Whether Claude Code acts on the output of a hook it killed is unestablished.
  - If it does not, the turn ends with no `Stop` line in `events.jsonl`, so `ParseEvents` never sees that turn end.
  - The Bound covers the mail but not this lost record.
  - The window is small, since `hookReadTimeout` is 10 s, but the record does not state it.

No finding on F3, F4, F5, F7, F8, F9 or F11.
- **F3:** checked against an idle strand, a working strand, a strand with no waiter and a strand mid-chain. Every case resets the count through a ring, a bell turn's `stop_hook_active` false, or a tool claim.

Tool calls made:
- `verbs.go:15-30` and a grep for `validateBody`: to see what `maxBodyBytes` guards and which verbs share it. Context showed only line 24.
- `verbs.go:85-96`: to read `Reply`'s validation, which context lacked.
- Three greps across `internal` for posts by program senders: to find posters outside creel. The first two found only `orchengine`; the third found LOOM's notice in `internal/loomcli/decision.go`.
- `internal/loomcli/decision.go:269-284`: to see whether LOOM's decision notice embeds unbounded text, which context lacked.
