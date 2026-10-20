---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D8 (Wake from lyx hibernate), D4
    summary: The wake reconcile's ls-remote and fetch are held by the network gate under the very hub pause entry the daemon lifts only after the reconcile, so the wake never completes.
    fix: In D8's wake bullet, state that the daemon's wake reconcile runs its network calls exempt from the hub pause entry its own request owns (for example by a gate exemption for that entry's owner, scoped to the reconcile), and state its bound - only the reconcile's two ls-remote calls and fetches pass, every other process stays held until the lift. Add to D8 that orch start and lyx cleanup under an operator's live hub pause wait at the gate until the lift, or exempt them the same way, and say which. Add an acceptance criterion - wake with a hub pause live completes the reconcile and lifts the entry.
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D4 (records branch lock order), D6 step 1
    summary: D4 waits on each records branch's write lock and push lock with no bound; at wake a push parked under the pause holds the weft's push lock while held at the gate, so the reconcile and the lift wait on each other, and D6's RefuseWhenBusy quiet wait is unstated.
    fix: In D4, replace the records-branch lock waits with one try-acquire each, as the prime lock already is - a held write or push lock reports the branch busy, judged and not moved, and its entry stopped per D7 at hub scope. In D6 step 1, state the quiet wait the teardown seam passes, zero, so a busy driver is reported busy at once. Extend D4's Bound line to name both - the reconcile never waits on a records write lock, a push lock or a driver's quiet wait. Add an acceptance criterion - wake with a parked push holding a paused pair's push lock reports that pair busy and lifts the hub entry.
  - id: F3
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D4 (rewound, rewritten), D3 step 2, D8 (lyx cleanup)
    summary: Rewound, rewritten and removed-from-origin are judged against the tracking ref as it stood before the fetch, but the cleanup dry run's own fetch and every other fetch (Fabric.Pull, the push retry, Bolt pull-first) move that ref first, after which a rewound branch reads ahead and the next push restores the dropped commits.
    fix: In D8's lyx cleanup bullet, make the judge-only reconcile fetch into no tracking ref (ls-remote SHAs plus a fetch of the objects only), so the dry run moves no ref, and say so as its bound. In D4, judge rewound and rewritten against the tracking ref's reflog, any earlier value of it, rather than its single pre-fetch value, so a fetch outside the reconcile cannot erase the evidence; add the reflog dependency to Constraints. Add an acceptance criterion - a task branch rewound on origin, then fetched by lyx cleanup without --apply or by lyx fabric pull, is still reported rewound by the next reconcile and its push does not restore the dropped commits.
  - id: F4
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D5 (Bolt replay by entry; On Bolt no entry change conflicts)
    summary: Re-applying a kept Bolt commit's entry changes through the board's write path meets its validation - the run lock on an entry origin now holds under a run status, a depends_on naming an entry origin removed, a cycle, an unknown label, and a new entry's id taken from nextID colliding with one origin added - so the claim that no entry change conflicts is false and the outcome is undefined.
    fix: In D5, state that the Bolt entry merge applies each kept change through the board's normal validation, never bypassing the run lock, and that a change the validation refuses is handled as the same-field clash is - origin's side kept, the local commit kept under its set-aside tag and reported with the refusal and the lost values. State that an entry the replay adds takes a fresh id from origin's board.json. Extend D5's Bound line to name both. Add an acceptance criterion - a local body edit of an entry origin holds under a run status is set aside and reported, and the replay does not stop.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D5 (drops one that becomes empty; Rationale)
    summary: On Bolt every re-render stamps README.md with the minute, so a commit whose entry changes origin already holds is never textually empty and is never dropped; and the rationale's premise that every lyx board write pulls Bolt first is false, since board writes save, render and spawn a detached board sync push, and only PullThenCommitWritten callers pull first.
    fix: In D5, define a Bolt commit as empty when its re-applied entry-level and design-file changes are empty, and state that such a commit is dropped with no re-render. In D5's Rationale, replace every lyx board write pulls Bolt first with every Bolt push meets the same replay.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D5 (caller list for the ended set)
    summary: D5 makes Bolt's seed-commit drop, which runs in pull-first on every PullThenCommitWritten write, a case of the replay helper, yet its list of callers and where each takes the ended set from names only the reconcile, Fabric.Pull and the push retry.
    fix: In D5's caller list, add Bolt's pull-first and state where it takes the ended set from - the rule of D3 step 2 over the origin Bolt tip its own fetch left, as the push retry does.
  - id: F7
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D10 (gone class), D5 (push-retry ended set)
    summary: The gone class exempts an ended entry by reading the ended set from the origin Bolt tracking ref, but the gone path is a failed push at once and fetches nothing, and a weft push's fetch need not move Bolt's tracking ref, so a parked push of a run landed elsewhere on a woken machine mails the orch as a failed push.
    fix: In D10's gone bullet, state that before judging ended membership the gone path fetches origin's Bolt head, one fetch of Bolt's branch, and reads the ended set from it; state in D5 that the push retry's fetch includes Bolt's branch for the same reason. Add to Acceptance criteria - a parked weft push of a run whose entry became done on origin since this machine's last Bolt fetch writes no failed-push record.
  - id: F8
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D8 (ended entry with a busy flag at wake)
    summary: The way forward wake names for an ended, busy pair - lyx batten pause then lyx cleanup --apply - is not shown to end the driver - batten pause takes effect at the next producer boundary, which a driver held by the pair pause entry does not reach, and outside a hub pause the cleanup reconcile still sees a live driver strand and reports the pair busy again.
    fix: In D8, give the ended-and-busy pair a way forward that ends its driver under the hold - lyx cleanup --apply's teardown of an entry in the ended set that wake gave a pair pause entry ends the driver without RefuseWhenBusy, as batten's own teardown does, since an ended entry's driver has nothing left to land - and state the bound - only ended entries holding a wake-written pair pause entry, every other pairteardown refusal still applies. Replace the lyx batten pause step in the notice with that. Add an acceptance criterion - after wake, lyx cleanup --apply removes the ended pair whose driver strand is live under its pair pause entry.
---

### [BLOCKING] Wake reconcile is held by its own hub pause

**Location:** `_lyx/discussion/decision-record.md` D8, Wake bullet; D4, the hub-pause exemption.

**Issue:** D8 has the daemon run the hub reconcile "after the wake and before it lifts its pause entry".
Every `lyx` process in a hub installs the network gate (`cmd/lyx/main.go` PersistentPreRunE calls `tendercli.InstallNetworkGate`), and the daemon is `lyx hibernate watch` spawned from the hub anchor (`internal/hibernatecli/spawn.go`).
`gitexec.runCore` sends every remote subcommand, `ls-remote` and `fetch` included (`internal/gitexec/transport.go`), through `pauserecord.Network`, which waits in `WaitUncovered` while a live entry covers the process, with no deadline of its own (`internal/pauserecord/doc.go`).
A hub entry covers every slug, and only a parent-lock holder is exempt, from slug entries alone.
So the reconcile's first ls-remote waits for the lift, and the lift waits for the reconcile.
The record decides the D4 liveness exemption for the hub pause but not the gate exemption the same pause imposes on the reconcile's own network calls.
The same gate holds `lyx orch start` and `lyx cleanup` under an operator's hub pause, which the record also leaves unstated.

### [BLOCKING] Records-branch lock waits are unbounded, and deadlock at wake

**Location:** `_lyx/discussion/decision-record.md` D4, "A records branch moves under its write lock, then its push lock"; D6 step 1.

**Issue:** D4 tries the prime lock once and never waits on a parent lock, but takes every records branch's write lock and push lock with a plain acquire, and its Bound line names no limit on those waits.
A detached push holds the weft's absorbing push lock for its whole loop (`coalescePush`, `coalesceFlagged` in `internal/fabricengine/coalesce.go`), and its `git push` waits at the network gate inside that loop while a pause covers it.
At wake the hub entry is live, so a push parked under it holds the paused pair's push lock until the lift; the wake reconcile, moving that pair's weft, waits on the push lock, and the lift waits on the reconcile.
D6 tears down through `RefuseWhenBusy`, whose existing caller pairs it with `RemoveQuietWait`, two minutes (`internal/pairteardown/teardown.go`), and the record names no wait for the reconcile's call, so each busy ended pair, which Open risks says is the usual case at wake, can add that wait while the hub stays paused.
The focus asked what bounds each wait; the record gives no bound for these.

### [BLOCKING] Rewound evidence is erased by any fetch outside the reconcile

**Location:** `_lyx/discussion/decision-record.md` D4 rewound and rewritten; D3 step 2; D8 `lyx cleanup`.

**Issue:** D4's rewound test is "origin head is an ancestor of the tracking ref as it stood before the fetch", and rewritten reads the same pre-fetch ref.
D8 has the judge-only cleanup run "both ls-remote calls and the fetches", which move the tracking refs.
After that fetch, or after `lyx fabric pull`, the push retry's fetch or Bolt's pull-first fetch, the tracking ref already sits at origin's rewound head; the next reconcile sees the local tip descend from it and classes the branch **ahead**, "the next push carries it", and that plain fast-forward push restores the commits `ResetPairCode` dropped, the harm D4's rewound outcome exists to stop.
The `lyx cleanup` dry run is documented as a listing of what `--apply` would remove (`internal/cleanupcli/doc.go`); its fetch here is the mutation that defeats the following `--apply`.
D3's removed-from-origin rule falls back to the merge base, so it survives; rewound and rewritten have no fallback.

### [BLOCKING] Bolt entry replay meets the board's write validation

**Location:** `_lyx/discussion/decision-record.md` D5, "On Bolt no entry change conflicts".

**Issue:** D5 re-applies each kept commit's entry changes "onto origin's `board.json` through the Bolt entry merge", and keeps origin's side only where both sides changed the same field or the local commit changes an entry origin removed.
The board's write path refuses more than that (`internal/boardengine/board.go` `boardCriticalSection`, `store.go` `validateWrite`):
the run lock refuses a net change to a run-held entry's scope, so a local body or priority edit re-applied onto an entry origin now holds under a run status is refused although the fields differ;
validation refuses a dangling `depends_on`, a cycle and an unknown label, so an added entry depending on one origin pruned is refused;
and a new entry's id comes from `nextIDIn`, the local maximum plus one, so two machines each adding an entry collide on id if the added entry is applied as it stood.
The record says neither what the replay does on such a refusal nor whether the merge bypasses the run lock; an unstated bypass is an unbounded guard bypass, and a stop is the "no way forward" D5's own rationale rules out.
On the focus's question, two machines each holding a status commit of a different entry do replay without a README or `board.json` stop; a commit changing another field of a held entry, or adding an entry, can still fail.

### [LOW] Bolt emptiness and the rationale's pull-first premise

**Location:** `_lyx/discussion/decision-record.md` D5, "drops one that becomes empty" and its Rationale.

**Issue:** `README.md` opens with the UTC date and time it was rendered (`internal/boardengine/doc.go`, README section), and the merge "re-renders `README.md`", so a re-applied commit whose entry changes origin already holds still carries a README diff and is never empty unless emptiness is judged on entry-level changes.
The rationale's "every `lyx board` write pulls Bolt first" is false: board writes save, render and spawn a detached `board sync` (`boardCriticalSection`, `Sync` in `internal/boardengine/sync.go`), and only the `PullThenCommitWritten` callers pull first.
The conclusion still holds through the push path, which D5 also replaces.

### [LOW] Pull-first is a replay caller missing from the ended-set list

**Location:** `_lyx/discussion/decision-record.md` D5, caller list.

**Issue:** D5 says "Bolt's push recovery and seed-commit drop become cases of it"; the seed-commit drop runs in `Bolt.pullFirst` (`internal/fabricengine/bolt.go`), reached from every `PullThenCommitWritten` call.
D5's per-caller statement of where the ended set comes from names the reconcile, `Fabric.Pull` and the push retry only, so the plan writer has no source for pull-first's set.

### [MEDIUM] Gone class reads the ended set from a Bolt ref nothing refreshed

**Location:** `_lyx/discussion/decision-record.md` D10, gone bullet; D5, the push-retry caller.

**Issue:** D10 exempts a gone branch of an ended entry from the failed-push record, reading the set "as D5's push-retry caller reads it", from the origin Bolt tracking ref "as their own fetch left it".
The gone class is a failed push at once with no fetch, and its only network step is an ls-remote of the pushed branch, so the Bolt tracking ref is as old as this machine's last Bolt fetch.
On a woken machine, the case the class was written for, a parked push of a pair whose run landed elsewhere since that fetch finds its branch gone, misses the exemption and mails the orch.
The weft push retry's fetch has the same gap if it fetches only the weft branch, which D10 leaves open.
On the focus's question, the gone class reads the branch's existence from its per-push ls-remote, and its ended membership from a ref neither path refreshes.

### [MEDIUM] Wake's way forward for an ended busy pair does not end its driver

**Location:** `_lyx/discussion/decision-record.md` D8, ended entry with a busy flag.

**Issue:** On the focus's question, the record never points an ended busy entry at `lyx resume`; that part is settled.
The way forward it names instead is `lyx batten pause <slug>` then `lyx cleanup --apply`.
`lyx batten pause` "request[s] a pause at a task worktree's next lifecycle producer boundary" (`internal/battencli/cli.go`), and the pair pause entry wake writes holds the driver's producer, so the record shows no path by which the boundary is reached while the entry stands.
`lyx cleanup --apply` runs outside a hub pause, so D4's hub-pause exemption does not apply, and pairteardown's quiet rule needs the driver strand absent, dead, retiring or parked (`internal/pairteardown/doc.go`); a live held driver reads busy again.
The record leaves the pair held and busy with no step that ends it short of `lyx resume`, which it forbids.

## Focus verdicts

Each directive not settled in a finding above is settled here.
D8 `lyx cleanup` without `--apply`: settled in F3, the judge-only mode still fetches, which is itself the defect.
Seams of D1, D4, D6 and D10: D1 names an injected seam for each of the teardown, the liveness probe, the notice and the Bolt merge, which settles how fabricengine reaches them; the lock-wait bounds are F2.
D10 no-orch episode: reported, by the next failure after an orch is recorded and by `lyx orch start` printing the outstanding record; the deferral writes a `deferred` record, ends the holder's loop and names `lyx fabric push`, so the holder neither loops nor goes silent.
D3 and D6 ended definition: an entry only local Bolt holds is absent from both the pre-fetch tracking ref and the merge base, so it is never ended; removed means absent from origin's tip but present in an origin commit local Bolt contains.
D5 reach, this machine's own landing: the Finalize-entered exclusion applies inside the helper for every caller, and batten run records stay in the prime weft until `lyx batten cleanup`, so the history it reads survives the pair's teardown.
