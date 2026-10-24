# Support log — idle-notice-no-live-run

## Interview

A darn run holds no interview; the writer read the board entry and the code and made each call itself.

## Rejected alternatives

- **Count a running record with a dead waiter as live when its strand is live.**
  Rejected: no record shows that this path caused the incident, and treating such a run as live changes Batten's quiet judgement on a guess.
- **Close the entry as a misquote with no change.**
  Rejected: the quoted text is specific to the zero-readings branch, so the quote is likely exact, and a silent skip would leave the next occurrence as unexplained as this one.
- **Change the notice text to count the skipped records.**
  Rejected: it widens `ReadAgentActivity`'s return for every caller to serve one diagnosis, when a trace line already reaches the investigator.
- **Reuse the unreadable-files episode set.**
  Rejected: its readable-again recovery for the same run directory would end the new episode on every poll.

## Review rounds

_No rounds yet._

## Question ledger

- Q: Which path hid the live run in the incident? — Open: the run directories and trace logs were on `osl-1046` and are gone from this machine; the prior task's investigation ruled out the fork reading, the attach pid rewrite and the run-directory root.
- Q: Which silent skip to instrument? — Self-picked: the running record whose waiter pid is dead, the one skip that can hide a working agent.
- Q: Which level? — Self-picked: Warn first, Debug while repeating, Info on recovery, per the logger's episode policy.
