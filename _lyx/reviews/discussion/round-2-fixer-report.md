# Fixer report — discussion, round 2

Every finding's premise held against the code; all eight are fixed in `_lyx/discussion/decision-record.md`, and the support log gains a Round 2 entry and ledger items Q24 and Q25.

## Changes

- **F1** (BLOCKING) — fixed as proposed.
  D8's wake bullet exempts the reconcile's network calls from the hub entry the daemon's own request owns, by passing the gate that entry's reference for the reconcile's duration, with the bound that only the ls-remote calls and fetches pass and every other process stays held.
  D8's orch start bullet states that orch start and `lyx cleanup` wait at the gate under an operator's hub pause.
  New acceptance criterion: wake with the hub pause live completes the reconcile and lifts the entry.
- **F2** (BLOCKING) — fixed as proposed, with one narrowing.
  D4 takes each records-branch write and push lock by one try-acquire, and a held lock reads busy.
  A busy pair weft stops its entry, but a busy Bolt, prime weft or the run scope's own weft stops nothing, because its holder's own push meets D10's retry; D3 states that a busy Bolt or prime weft is no D3 stop.
  The narrowing keeps Merge-In and Finalize from stopping on their own run's detached push (self-check, below).
  D4's Bound names the records locks, the push lock and the quiet wait, and D6 step 1 sets a quiet wait of zero.
  D8 names `lyx resume <slug>` as the way forward of a pair stopped busy by a parked push.
  New acceptance criterion: a parked push holding a paused pair's push lock reports that pair busy and the hub entry lifts.
- **F3** (BLOCKING) — fixed as proposed.
  D4's rewound and rewritten tests read the tracking ref's reflog from the last entry this machine's own push wrote onward, and rewound takes precedence over ahead.
  Limiting the window to entries since the last push keeps the `ours`-merge way forward and this machine's own leased rewind from tripping the test.
  D8's cleanup dry run judges against the ls-remote heads and fetches objects into no ref (`--refmap= --no-write-fetch-head`), with the bound that it moves no ref.
  A Constraints line names the reflog dependency and its expiry.
  New acceptance criterion as proposed.
- **F4** (BLOCKING) — fixed as proposed.
  D5 applies each Bolt change through the board's validation with the run lock in force, handles a refused change as a field clash is handled, and gives an added entry a fresh id from origin's `board.json`.
  The Bound and a new acceptance criterion name these rules.
- **F5** (LOW) — fixed as proposed: a Bolt commit is empty when its re-applied entry-level and design-file changes are empty, and it is dropped with no re-render. The rationale now reads "every Bolt push meets the same replay".
- **F6** (LOW) — fixed as proposed: pull-first is in D5's caller list, reading the ended set from its own Bolt fetch.
- **F7** (MEDIUM) — fixed as proposed: D10's gone path fetches Bolt's branch before judging ended membership, and D5 and D10's retry bullet state that the push retry's fetch includes Bolt's branch. New acceptance criterion as proposed.
- **F8** (MEDIUM) — fixed as proposed, with one extension.
  As proposed, D6 has `lyx cleanup --apply` tear down an ended entry under a wake-written pair pause entry (caller `hibernate`) without `RefuseWhenBusy`, and the Bound limits this to that case; D8's notice drops `lyx batten pause`.
  The extension: once the driver is ended, the teardown lifts the pair entry so that a parked push can finish, and a second `lyx cleanup --apply` then removes the pair.
  The Open risks line is updated to match, and a new acceptance criterion was added as proposed.

## Self-check

- fix-consistency, MEDIUM — the busy rule stopped the run scope's own pair, unlike the live rule: fixed, the run scope's own weft is exempt from the busy stop.
- fix-consistency, MEDIUM — D6 Bound's first sentence was left unqualified: fixed, "outside the one case above".
- fix-consistency, LOW — D3 did not exclude a busy Bolt or prime weft from its stops: fixed, D3 says it is no such stop.
- fix-consistency, LOW — D10's retry bullet did not mention the Bolt fetch: fixed.
- fix-claims, BLOCKING — the run's own detached push makes Merge-In and Finalize stop: duplicate of the first fix-consistency finding, fixed there.
- fix-way-forward, BLOCKING — the same run-scope busy stop with no way forward: duplicate, fixed there.
- fix-way-forward, MEDIUM — an ended pair kept busy by a parked push under the wake entry had no allowed way forward: fixed by the F8 extension above.
- fix-way-forward, LOW — the gone path's Bolt fetch had no failure class: fixed, its transport failure is the push's own.
- fix-way-forward, LOW — "`lyx board` can write again" cannot be carried out while a run lock holds: fixed, after the run holding the entry ends.
- fix-proposal, MEDIUM — after a wake-time D3 stop, orch start waits at the gate behind the kept hub entry: fixed, the D3-stop notice orders `lyx resume` before any `lyx orch start` or `lyx cleanup`.
- fix-proposal, outside its lens — a Bolt commit touching only done entries, such as a prune, is set aside: not a fix-check finding, since it predates this round and the fix did not reach it; left for the next review.

## Disputed

None.

## Deferred

None.
