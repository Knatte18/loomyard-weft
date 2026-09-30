Today the ly-drive skill never repairs a blocked run: it writes a report, hands back and exits, and every recovery is done by the hub by hand (a message the operator must also approve in the pane, since the driver runs in bypass mode).
On 2026-09-30 most hand-backs were transient or false stops that a single re-step after the underlying fix would have cleared.

Wanted:
- A closed list of safe repairs the driver may do on its own, each recorded under `<scratch>/repairs/`: one re-step of a stop whose reason matches a known-transient class (flaky verify once `verify-triage` exists, a verb that failed on infrastructure), and a re-step after the lyx binary changed since the stop (a deploy landed a fix).
- Everything else still hands back; `awaiting` at Publish is the operator's hand-off, never auto-approved.
- The driver stays alive after a hand-back (waits for a resume message) instead of exiting, so the hub resumes it with one message rather than `lyx loom start` spawning a fresh session.