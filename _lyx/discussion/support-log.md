# Support log — smokestart-index-lock-flake

## Interview

A darn run holds no interview; the board entry is the whole statement of the task.

## Rejected alternatives

- Waiting with `waitRecordsIndexLockGone` between the test's starts: it narrows the window only in one test, and the first start's driver keeps probing the weft after the wait returns, so the race stays live there and in production.
- Retrying the start's records commit on `index.lock: File exists`: it adds a retry loop at one commit site while every other locked writer stays exposed to the same probe.
- Taking the weft write lock around every status probe: it touches many call sites across packages and makes read probes contend with writers for no gain over not locking at all.

## Review rounds

_No rounds yet._

## Question ledger

- Which process held the index? Self-picked: an unlocked read-only `git status` probe, because every records commit site takes the weft write lock and `git status` takes `index.lock` opportunistically.
- Remote git children too? Self-picked: yes, one environment for every child keeps the rule without exceptions, and a remote command never needs an optional lock.
