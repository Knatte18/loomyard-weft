---
round: 4
focus:
  - "Hinted channel reads in `_lym/discussion/decision-record.md` D2 (and D1's acknowledgement of envelopes): the relay removes a hint before the read it triggers, and the holder has already acknowledged the Slack event. Settle what re-triggers a read that fails on a transport error, a server error or a rate limit, with no further message and no reconnect, and whether the Decision and its acceptance criteria state that retry and its bound."
  - "Re-ensure of a held channel in D5: D5 says a refused operator invite is retried at that channel's next ensure, and the ensure list gives the prime's channel at relay start only, a beam's at its first routed mail, and listed beams at relay start or a presence flip. Settle which event runs the next ensure for a channel whose entry records another operator than setup.json's or none, whether `lym slack setup --operator` takes effect with the relay running, and whether the acceptance criteria name that event."
  - "A failing `hubregistry.Live` read in D1 and D3: events stay unacknowledged and ensures fail closed with an orch notice. Settle what a holder with no ensure to make does while the read keeps failing, who learns of it, and whether the Decision states that notice and its bound."
---

Focus targets D1, D2, D3 and D5 of the decision record, where the relay's recovery paths after a failure are specified.
