---
verdict: APPROVED
prior:
  - id: F19
    status: resolved
    answer: "D8 now sends a run whose work the parent holds, whatever its entry says, to a held path that never merges the parent into the task worktree and runs no gate, the `landed` bullet takes the same path, and the D8 criterion says so."
  - id: F20
    status: resolved
    answer: "Accepted as fixed differently. D6 gives `Deps.RecordLanding`, the size and empty checks and `Deps.LandLocally` the run's own landing commit; `runpusher.RecordLanding` rewrites the landing by slug and `AheadByOnly` reads the tips as a set, so the same tip keeps the parent ahead by owed tips alone. Both criteria are in the record."
---

No new finding reaches BLOCKING on a changed hunk.
Both prior findings are resolved.
Everything below was settled by reading code and the record; nothing was built, run or driven.

## Focus verdicts

- **A member the leader landed (D8).**
  Under the lock it merges nothing into its task worktree and runs no landing gate.
  It judges the reconcile's ended and stopped verdicts and `handedOverStop`, and skips the land check and the parent sync.
  With the queue entry missing, its landing commit is the newest `Lym-Landing` commit naming it, else the parent's head.
  Two MEDIUM weaknesses in this path are listed below.
- **Remaining landing records (D6, D8).**
  The push ledger and the dobby's record get each run's own landing commit, the leader's included.
  A landing written again keeps its tip, so the next lock holder reads the parent as ahead by owed tips alone.
  A landing the parent's pusher already settled and removed is written again with a tip origin holds,
  which the next pusher pass settles once more.
- **A rewritten member with a lost entry.**
  Yes, it can be landed a second time.
  `HoldsWork` reads it as not held, and the record reads the `Lym-Landing` trailer only to name the commit of a run already judged held.
  Inside one chain, a later entry that changes the same lines differently should conflict and be left out,
  unless its branch carries the earlier entry's commits.
  The ordinary route is a later landing between the move and the member's wake-up.
  This stays MEDIUM: it needs both that rewrite and a lost entry.

## Not raised: below the incremental bar

These are MEDIUM or LOW, so this incremental round does not raise them as findings.
The parent or a later round may want them.

- MEDIUM, changed text.
  The `landed` bullet is reached only when the parent does not read as holding the run's work.
  Its held path then runs the run-scope reconcile.
  The parent's pusher will usually have marked the entry done by then.
  `readOriginEnded` drops a run at Finalize from the ended set only while `HoldsWork` reads it as held,
  or once its history names Finalize (`finalizeEnteredRead`, internal/fabricengine/replay.go).
  A first-entry member can therefore halt as "origin ended this run", with a way forward naming `lym cleanup --apply`.
- MEDIUM, changed text.
  D8's last fallback records the parent's head as not empty, "as a held run with no merged parent head records it today".
  Today a held run under the lock still runs `mergeAndGate`, which reads the parent head, so a landing that changed
  nothing is recorded as empty (internal/landingshed/finalize.go:526-529, 1089-1095).
  Under D8, a run whose branch adds nothing and has no `Lym-Landing` commit reaches the dobby's notice as
  "landed as" another run's commit.
- MEDIUM, second route to a second landing.
  A resumed member with a `landed` entry, whose lines a later landing rewrote, meets D1's lock-free merge first.
  D1 states no skip for it.
  The merge moves its task tip, and D2 then drops the `landed` outcome.
- MEDIUM.
  The leader's own entry can conflict in the chain once the parent has moved.
  D4 gives it `conflict`, but no Decision says whether the leader goes on to land the other entries or frees the lock.
- MEDIUM.
  D6's parent move names one guard, the head at the chain base.
  `Fabric.Merge`, which it replaces for a batch, also refuses a dirty checkout, a merge in progress,
  foreign merge state and a detached HEAD, and takes the weft write lock.
  The sync's dirt probe runs before the gates, so it does not cover the time the gates take.
- LOW.
  D6 says a member's records commit is "the oldest one origin lacks, as `oldestHeldRecords` picks it today".
  `oldestHeldRecords` returns the commit it is given unless a standing landing holds an earlier one;
  the oldest-unpushed read is `OldestUnpushedRecords`.
- LOW.
  D6 has the leader write a ledger landing for every entry and does not except pushes off,
  where `pushParent` records none today.
