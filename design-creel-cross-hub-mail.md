# Creel mail between hubs with a verified sender and one-way hub trust

`creel-cross-hub-mail` · Task · sketch, creel, orch · running · Open-Window

## Seen (operator, 2026-10-30)

The operator sometimes has the orch write straight into another repository, such as Scribe or Quarry, and would rather let the orchs of the hubs talk to each other, which is how lym is meant to be used.
Creel has no support for this today.
One daemon serves one hub, its socket is named from the hub path, and the hub a call reaches is the hub of the directory it runs from.
A process outside the hub's strands is `EXTERNAL`, shown with a terminal and parent-process envelope, and its replies reach only a waiting `post --wait-reply`.
`EXTERNAL` is the wrong sender for this: the receiver must see the real strand that sent the message.

Seen again 2026-11-05: the loomyard orch could not steer trial run `codex-greet` in the sandbox hub `lyt` out of PR-Review, since lyt's reviewer and `lyt:orch` refused its mail as `EXTERNAL`, so the operator was asked to type in a sandbox hub's pane, which they never want to do.

## Verifying the sender (operator, 2026-10-30)

Any process of the same user can claim any sender, so a prompt-injected agent could claim to be `ly:orch`.
The sending daemon, which reads the caller from `SO_PEERCRED` as it does today, vouches for what it sent, and the receiving daemon asks it to confirm:

1. The receiver derives the sending hub's socket name from its hub path, as `creel-<shortname>-<hash>.sock`.
2. The receiver sends one call in creel's JSON-lines protocol, such as `confirm`, with the message id, the receiver and a hash of the text.
3. The sender's daemon looks in its own journal for a message from that strand to that receiver with that hash, and answers yes or no, never the text.
4. Only on yes does the receiver show the sender as that strand; otherwise the message shows as unverified.

This is a call and not a signature, since a signature needs a secret and every agent of the same user can read a secret file.
A message can be confirmed once, so a replayed message is refused.
The receiver confirms when the message arrives, within the journal's retention.
A sending daemon that is down is started as a client starts it today, or the message stays unverified.
The check works only within one user, since the sockets do.

## Proposals (orch, agreed by the operator 2026-11-05)

Starting points for the Discussion, not final:

- Transport: the sending daemon delivers straight to the receiving hub's socket, then the receiver confirms as above; an inbox folder (the operator's first idea, 2026-10-30) is needed only if a receiver cannot be started, so the Discussion weighs whether to keep it at all.
- Addressing by hub shortname, such as `lyt:orch` or `cs:orch`, through a record on the machine that maps a shortname to a hub path, written when a hub is created or opened.
- Replies: `lym creel post --reply <id>` works across the hub border by the same route, back to the strand that sent.
- Trust per hub, in the hub's config, such as `trusted_hubs: [ly]`:
  - Verified mail from the orch of a trusted hub carries the authority mail from the hub's own orch carries today.
  - Verified mail from any other strand, or from a hub not listed, is data to weigh, never an instruction.
  - Unverified mail shows as unverified and is data only.
- Trust runs one way: loomyard steers the sandbox hubs `lyt` and `cs`, which list `ly`; loomyard lists neither, since the sandboxes run untrusted code and Codex and a prompt-injected sandbox orch must gain no hold on the main repository.
  The sender check stops a forged sender; it cannot stop a trusted orch that is itself prompt-injected, which is why trust is not mutual.
- Roles and the creel note in the shuttle hooks learn the new sender kinds, so an agent obeys verified trusted-orch mail and refuses the rest as today.

## Not in this round

- Another machine: a synced directory reaching a hub on Neve.
- How this fits a closed area for lym's process tree (research note `sandboxing-options-2026-10-30`).

## Trial at the end

A darn `docs` task in which the loomyard orch drives a run in `lyt` from start to landing through creel alone, the operator doing nothing, and records the result in `docs/research/`.
