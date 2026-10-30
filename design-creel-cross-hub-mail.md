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

## Open design points

- Whether a file is enough to name the sender: any process of the same user can write a file, so a file's sender field can be forged by a prompt-injected agent.
  One way: the sending daemon, which reads the caller from `SO_PEERCRED` as it does today, writes the file, and the receiving daemon asks the sending hub's daemon through its socket to confirm the message before showing the sender as that strand; an unconfirmed file shows as unverified.
- Where the folder lives and how a sender addresses a hub: a path derived from the hub path, like the socket, and a record on the machine that maps a shortname to a hub path.
- What a reply does across the hub border: it goes to the sender's inbox folder, addressed to the agent the file names.
- How a role treats mail from another hub: as data to weigh, never as the operator's instruction, since it is a channel for prompt injection into the receiving orch.
- How this fits a closed area for lym's process tree (research note `sandboxing-options-2026-10-30`): the folder sits outside an area, so reading and writing it goes through the broker.
- Another machine: a folder that a synced directory carries would reach a hub on Neve; not designed here.
