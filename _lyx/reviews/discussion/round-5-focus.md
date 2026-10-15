---
round: 5
focus:
  - "Wiring versus verbs that spawn nothing: in `decision-record.md`, do D6 and D7 give one consistent rule for whether `webstercli`'s and loom's wiring (run once per invocation for every verb) can refuse a verb because of the Master `master` key? Settle whether `record-batch`, `reset`, `validate`, an attach, an await and loom `status` and `pause` can be refused by that key, and whether an acceptance criterion pins it."
  - "Check placement in `websterengine.Run` and in the recover-batch spawn branch: does D6 pin each wrong-key refusal ahead of the first state mutation (entry-time strand reclaim, `--fresh` archiving, stale report archiving, stopping a prior recovery strand), and do the acceptance criteria say a refused `run` or `recover-batch` leaves state and live strands unchanged?"
  - "Recovery session engine reads: does D6 say which `recoveryEngines` entry (keyed by session id) serves the turn-end read of a live recovery record, and what a wrong-engine read of an engine-less entry does when the engine requires the session's transcript, with a way forward for any refusal?"
---

Focus entries target the fixed sites only; the reviewer still reads the whole pair.
