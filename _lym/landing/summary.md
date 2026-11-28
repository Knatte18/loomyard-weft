# Gate slot waiters take a priority: parent lock holders first

Gate slot waiters now have a priority, so a free slot goes to the waiter that most needs it.
Until now every waiter polled on equal terms.
A run gating its landing while it holds the parent lock waited for the same slots as the runs queued behind that lock.
Those runs' lock-free gates go stale anyway once the landing moves the parent.
`docs/research/webster/webster-time-2026-11-08.md` (lever 4) measured 3.1 min of slot wait per Webster row on average, and 27 min in one row.

## What changes

- `gateslot.Priority` ranks a wait.
  `PriorityLockHolder` goes first, then `PriorityImplementer`, then `PriorityOther`, the zero value.
  `gateslot.SlotPriority(key, holdsParentLock)` derives it.
  A call whose run holds a parent lock is a lock holder whatever its gate key.
  A `card` or `batch` call is an implementer, and every other call is `PriorityOther`.
- `Holder.Priority` carries the rank into `Acquire` and `AcquireSlots`, and the holder record stores it as `priority`.
- When a waiter above `PriorityOther` starts waiting, it locks a queue ticket at `<slot dir>/queue/<priority>-<pid>-<n>.lock`.
  It removes the ticket once its acquire returns.
  While a higher ticket's lock is held, a lower acquirer takes no slot and no drain lock, even when a slot is free.
- A lower multi-slot acquire can hold the drain lock and only part of its slots when a higher waiter shows up.
  It then releases its slots and the drain lock and waits again, so the higher single-slot waiter is not stuck behind the drain.
- `lym gate test` sets the priority once per call, before its first tier.
  It asks the hub's parent lock store whether a live lock is owned by the call's worktree slug, and the gate key covers the rest.

## Design decisions

- Liveness comes from an OS file lock on the ticket, the same way the slots and the drain already work.
  A crashed waiter's lock goes when the process dies, so its leftover file outranks nobody.
  A probe deletes an unheld ticket file only while it holds that file's lock, and only once the file is older than a minute.
  A younger file may belong to a waiter that has created it and not locked it yet.
- A held slot is never taken back.
  Priority decides only who takes a slot that frees, and who may hold a partly filled drain.
- `PriorityOther` writes no ticket, since no waiter ranks below it.
  The non-tier holders keep the lowest rank: a card's `**Verify:**` run, a drift command, the plan lint's type load and `cmd/testtiming`.
- Waiters of one priority still poll in no fixed order.
  Steady demand from a higher rank can starve a lower one, which is the intended order.
- A parent lock store that cannot be read is logged, and the call waits as a run holding no lock.

## Tests

`internal/gateslot/priority_test.go` covers:

- the priority table of `SlotPriority`;
- a lock holder taking a freed slot ahead of an implementer that waited first, with the implementer's turn coming after;
- a lower multi-slot acquire giving up its partial slots and drain lock to a higher single-slot waiter, then finishing once the slots free;
- an unheld ticket outranking nobody, with a stale one removed and a fresh one kept.

No test covers the `gatecli` wiring that reads the parent lock store, `holdsParentLock`.
Exercising it would take a hub fixture with a live lock owner around a full `lym gate test` call.

## Merge danger

**Door:** two-way
**Blast radius:** gate
