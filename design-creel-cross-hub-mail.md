# Creel mail between hubs through an inbox folder, with a verified sender

`creel-cross-hub-mail` · Note · feature, creel, orch

## Seen (operator, 2026-10-30)

The operator sometimes has the orch write straight into another repository, such as Scribe or Quarry, and would rather let the orchs of the hubs talk to each other, which is how lym is meant to be used.
Creel has no support for this today.
One daemon serves one hub, its socket is named from the hub path, and the hub a call reaches is the hub of the directory it runs from.
A process outside the hub's strands is `EXTERNAL`, shown with a terminal and parent-process envelope, and its replies reach only a waiting `post --wait-reply`.
`EXTERNAL` is the wrong sender for this: the receiver must see the real strand that sent the message.

## Idea (operator, 2026-10-30)

A shared inbox folder takes in messages from outside the hub.
The file a message becomes carries its sender: the path of the sender hub's inbox folder and the name of the agent that sent it.
A message needs a receiver, so the daemon knows whom to give it to.
A reply is addressed from those two fields, so the replier knows which agent in which hub to answer.

## Verifying the sender (operator, 2026-10-30)

A file cannot prove its sender: any process of the same user can write one, so a prompt-injected agent could write `ly:orch` in the sender field.
The sending daemon, which reads the caller from `SO_PEERCRED` as it does today, writes the file.
The receiving daemon then asks the sending hub's daemon to confirm it:

1. The file names the sender hub's path, and the receiver derives that hub's socket name from it, as `creel-<shortname>-<hash>.sock`, with no registry.
2. The receiver sends one call in creel's JSON-lines protocol, such as `confirm`, with the message id, the receiver and a hash of the text.
3. The sender's daemon looks in its own journal for a message from that strand to that receiver with that hash, and answers yes or no, never the text.
4. Only on yes does the receiver show the sender as that strand; otherwise the message shows as unverified.

This is a call and not a signature on the file, since a signature needs a secret and every agent of the same user can read a secret file; the sending daemon cannot be fooled, because it reads its caller from the process and knows what it sent.
A message can be confirmed once, so a copied file cannot be reused.
The journal's retention bounds how late a confirmation can come, so the receiver confirms when the message arrives.
A sending daemon that is down is started as a client starts it today, or the message stays unverified.
The check works only within one user, since the sockets do.

## Open design points

- Where the folder lives and how a sender addresses a hub: a path derived from the hub path, like the socket, and a record on the machine that maps a shortname to a hub path.
- What a reply does across the hub border: it goes to the sender's inbox folder, addressed to the agent the file names.
- How a role treats mail from another hub: as data to weigh, never as the operator's instruction, since it is a channel for prompt injection into the receiving orch.
- How this fits a closed area for lym's process tree (research note `sandboxing-options-2026-10-30`): the folder sits outside an area, so reading and writing it goes through the broker.
- Another machine: a folder that a synced directory carries would reach a hub on Neve; not designed here.
