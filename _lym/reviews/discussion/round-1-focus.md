---
round: 1
focus:
  - "D1 and D7 remove guards from the transition (network-gate admission `pauserecord.Admit`, the pre-push origin reconcile, `push_pending` forcing a push): does each Decision state what the run can now do or skip, and what bounds it, including the window before the pusher's reconcile first runs?"
  - "D4, D3 and D6 rest on tool and code facts (`retryWeftPush` try-acquires the weft write lock; a rejected records push leads fabric to read the takeover and call `pairteardown.HandOver`; a board status write already spawns a detached sync): is each stated with its observed result and where it is pinned, or only asserted?"
  - "D3's lock and exit protocol, D5's restart paths and D8's claim that `runfsm`'s push-owed input reads no owed state once owed retries write no failed-push record: does any case leave a commit unpushed, start two pushers for one worktree, or hold the run, and does each Decision's bound cover it?"
  - "D5's refused-push stop and D6's hand-over: does any Decision leave a fact the design rests on for Plan or an implementer to find out (for example which worktree kinds `HandOver` removes, or how the pusher outside the pair resolves its records worktree)?"
  - "Decisions versus Constraints, Auto-mode assumptions and Scope: is any requirement stated twice or in conflict (for example the Worktree-Teardown wait in D5 against D1's no-wait rule), and does any Decision or PATTERN change carry deliberation that belongs in `support-log.md` rather than `decision-record.md`?"
---

Seed focus for round 1 of the discussion review.
Entries name where to look; none caps severity or pre-states a verdict.
