# Support log — board-push-race

## Interview

A darn run holds no interview; the decision record rests on the board entry and a reading of the code.

## Rejected alternatives

- Make every `Bolt.PushRecorded` take `board.push.lock`: `Bolt.Sync` holds that lock around its own push, and a second acquire on a new file description blocks, so board sync would deadlock.
- Hold `board.lock` from commit through push: every board write would wait on the network, which the board's write path promises never to do.
- Retry longer instead of judging: a longer backoff narrows the race but still records whatever the last attempt failed with, so the false divergence remains possible.
- A new failure class for a non-divergent rejection: `other` already carries git's error and a plain-retry way forward, and the class set is pinned by tests and the status verbs.

## Review rounds

_No rounds yet._

## Question ledger

- Which two processes pushed the board at 15:22 on 2026-10-30? Self-picked: not identifiable by pid, since the hub's logs and creel journal for that day are pruned; the record names the candidates from the code (D4).
- Should commit and push share one lock? Self-picked: no (D4).
- Should a non-divergent rejection that never lands still mail? Self-picked: yes, under `other` (D2).
- Should the judgment run on the code and records sides too, or only Bolt? Self-picked: every side, since the classification is shared (D1).
