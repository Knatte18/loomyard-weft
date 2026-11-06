---
format: 5
approved: true
language: go
---

# Plan: creel mail across the hubs of one machine

A strand of one hub mails a strand of another hub on the same machine through creel, and the receiver sees the real sending strand, confirmed by the sending hub's daemon.
A hub obeys such mail only from the orch of a hub it lists as trusted; all other mail from another hub is data to weigh.
The cards build upward: the machine record and the `creel` config, the journal and stamp that hold cross-hub mail, the seam that reaches another hub, then post, receive, delivery and replies, and last the agent instructions and the trial note.

## Card Index

1 — hub-registry — add the machine record of hubs and the per-user state directory it sits in
2 — creel-config — add the hub-wide creel config module with trusted_hubs
3 — cross-hub-records — fold and render outbound, inbound and dropped records and the sender stamp
4 — hub-directory — add the told HubDirectory seam and record the hub at daemon start
5 — hubs-verb — add lym creel hubs
6 — post-to-another-hub — queue a strand's post to another hub and refuse the other verbs
7 — receive-and-confirm — serve deliver and confirm with drops, notices and trust
8 — outbound-worker — deliver the queue with retries, the start cap and the hour's return
9 — replies-across-hubs — carry replies back and settle forward, sweep and chase for such mail
10 — agent-instructions-and-trial — make the instructions read the stamps and file the trial note

## Shared Decisions

### Decision: trust-sentence-home

- **Decision:** D7 places the creel trust sentence in `internal/shuttleengine/claudeengine/settings.go`, which only reads it; card 10 changes the constant `CreelTrustSentence` in `internal/shuttleengine/creelhooks.go`, where it is defined.
- **Rationale:** the sentence is one constant Claude and Codex share, so changing it there is what D7 asks for, and no copy exists to diverge.
- **Applies to:** card 10

### Decision: nil-hub-directory

- **Decision:** a nil `HubDirectory` means a daemon that serves no other hub, so a hand-built store or rig keeps today's behaviour; production always passes a seam, whose calls answer the cause when the per-user state directory cannot be read.
- **Rationale:** every existing store and daemon test stays as it is, and only the rows that name another hub set the seam.
- **Applies to:** cards 4 to 9

### Decision: d1-way-forward-holds-for-a-live-daemon

- **Decision:** beside its start, a daemon records its hub on a served local verb: at most once a minute after a record that succeeded, and at the next such verb after one that failed.
- **Rationale:** D1 names `lym creel inbox` in that hub as the way forward, and a verb against a live daemon starts none, so a lost entry would otherwise stay lost until the daemon restarts.
- **Applies to:** card 4

### Decision: d1-none-refusal-names-inbox-alone

- **Decision:** D1's none refusal names `lym creel inbox` alone, not D1's "any `lym creel` verb".
- **Rationale:** `lym creel hubs` starts no daemon and writes no record, so running it in a hub records nothing; D1's acceptance criterion needs only `lym creel inbox` named.
- **Applies to:** card 1

### Decision: d2-unusable-answer-is-a-code

- **Decision:** an answer the sending daemon cannot use (undecodable, for a foreign hub path, or with neither `accepted` nor `refused`) gives the code `bad-answer`, though D2 quotes an answer of the receiving daemon.
- **Rationale:** such an answer is not known to come from the receiving daemon, or carries no reason to quote, and the return notice is `CREEL` mail an agent follows, so none of its text reaches the notice.
- **Applies to:** card 8

### Decision: no-drop-for-a-held-message

- **Decision:** a confirm that answers no is checked against the held messages under the store lock, and a matching held message answers `accepted`, with no drop.
- **Rationale:** an overlapping retry can store a message and journal its remote delivery while an older attempt's confirm is in flight; D3's rule that a held message answers its id, and the operator's rule that a genuine message is never dropped, both call for it.
- **Applies to:** card 7

### Decision: outbound-outside-the-pool

- **Decision:** an outbound message lives in its own map of the folded pool, never in `messages`.
- **Rationale:** the flags, workers, reminders, sweeps, chases and settles all read `messages`, so D2's "held by no local strand" holds by construction instead of by a guard at each site.
- **Applies to:** cards 3, 6, 8 and 9

### Decision: deploy-note

- **Decision:** every session of the machine's hubs launched before the deploy restarts right after it, and until it does an orch strand of another hub reaches it under the old trust sentence (D7, D9).
- **Rationale:** the design bounds that exposure by rollout; the board note of card 10 carries the same setup for the trial.
- **Applies to:** the landing of this task

## PATTERN changes

- `PATTERN-agent-addressing` — changes: Naming or messaging an agent: names only via `internal/agentname`; messages only through creel; across hubs daemon to daemon with the sender confirmed, authority only for a trusted hub's orch.
