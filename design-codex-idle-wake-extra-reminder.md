# A Codex strand with a wake queued also gets a Mail-is-waiting reminder turn

`codex-idle-wake-extra-reminder` · Task · bug, codex, creel, shuttle · running · Describe

Seen in the live creel trial in lyt (docs/research/codex/engine-trial/codex-creel-trial-2026-11-05.md, burst case). Three mails posted back to back to one idle Codex strand: the first is read by the idle waiter and queued through codex queue; the next two arrive while that queued turn has not started yet. Creel then types the pane-path line "[creel] Mail is waiting for you. End this turn now; creel hands the mail to you as the turn ends. Do not run lym creel read for it." into the composer. That line starts a turn of its own (a wasted model turn), the Stop hook of that turn delivers the other two mails, and the queued first mail starts only after it: it reached the model 17 s after the post, where the same burst to the other session took 8 s.

Expected: a strand whose wake is queued, and whose waiter has read its mail, gets no reminder line, since the queued turn and the Stop hook deliver the rest.

The model answered every mail and no mail was lost or crossed to the other strand; the cost is one extra turn and the delay.