```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I checked pairs of decisions that act on the same state, then checked each criterion and constraint against the decisions it touches.

**Pairs checked**
- D8 held-work-first rule × D8 `landed` bullet × D6 landed-size rule × the D6/D8 criterion: **broken** (F-a)
- D1 lock-free merge rationale × D3 batch of one / D6 "lands alone" fallback / D8 `failed` own path: **broken** (F-b)
- D3 batch take × D6 pre-move member checks and rebuild: **broken** (F-c)
- D6 rebuild after a left-out member × D7 culprit and the entries after it: **broken** (F-d)
- D4 `conflict` outcome × D3 "own entry first" × D8 `conflict` bullet, when the leader's own entry conflicts with the synced base: **broken** (F-e)
- D3 cap × D9 cap × the D3 criterion: **broken**, wording only (F-f)
- D2 entry lifecycle × D8 `failed`/`conflict` kept until it lands alone: held
- D2 dead-entry pruning × D6 backstop ordering × D8 backstop-first: held
- D2 rewrite at rejoin × D8 "tip past entry head" clause: held
- D5 marker across bisection steps × D7 no new chain: held
- D5 restore × D7 leader-culprit merge of the chain base after the last restore: held
- D5 resume restore × D8 wake paths, where a member's worktree is never detached: held
- D6 pre-move re-read of pause, liveness and tip × D8 `landed` needing tip equal to the entry head: held
- D6 backstop setting `landed` × D8 `landed` tip check: held
- D6 leader-written ledger landing × D8 "writes its ledger landing if missing": held
- D7 batch gate without a recorder × D7 leader culprit writing the failure record from the prefix-1 log: held
- D7 culprit `failed` × D2 keep-until-alone × D3 skipping entries with an outcome: held
- D9 `batch_max` 1 × D3 batch of one: held
- D10 run with no entry × D8 "no outcome leads a batch" × D10 "own PR decision lands alone": held, provided the PR decision precedes the batch take as today (`pullRequestLanding` runs right after the lock)
- D1 removal × D9 key removal: held
- Constraint on the idle release × D5/D7 gates (`finalizeKeepsLock` keeps the lock while the status names Finalize): held
- Constraint on `LandLocally` arguments × D6 `oldestHeldRecords` pick from the member's pair path: held
- Criteria D7 "batch of four, third fails" and the D9 claim "two extra gates": held (prefixes 4, 2, 3)

**Findings**

F-a — BLOCKING, design, `_lym/discussion/decision-record.md:152`
- **Summary:** D8 sends every run whose work the parent holds to "today's already-landed path, whatever its outcome says". A member the leader landed always reads as held, so D8's `landed` bullet is never reached. Each landed member then merges the parent and runs a full gate under the lock. That contradicts the `landed` bullet, the D6/D8 criterion ("runs no merge and no gate") and the Goal of one gate per batch. It also records the dobby's landing as the parent head with an empty diff, against D6's first-parent measurement.
- **Evidence:**
  - `internal/landingshed/finalize.go:473-528`: with `held` true, `landUnderLock` still calls `mergeAndGate`.
  - `internal/landingshed/finalize.go:960-994`: `recordLanding` uses the parent `HeadSHA` and the `parentHeadMerged` read just before that merge.
  - `internal/loomcli/landingdeps.go:270`: `VerifyDir` is `verifytree.Dir(l.AnchorPath())`, one per worktree, so the leader's pass does not let the member skip the gate.
  - `internal/fabricengine/parenttips.go:225`: `HoldsWork` holds after a squash.
- **Fix:** in D8, route a run whose entry is `landed` with its tip equal to the entry head by the `landed` bullet before the held-work test. Keep "today's already-landed path" for a run with no `landed` outcome. Alternatively, say that the held path for a queued run merges and gates nothing.

F-b — MEDIUM, design, `_lym/discussion/decision-record.md:65`
- **Summary:** D1's rationale says a conflict session never holds the lock, and D8 frees the lock for a conflicting merge only on a `conflict` run. A batch of one (D3), a leader past the rebuild bound landing alone (D6:114) and a `failed` run's own path (D8:158) all run today's merge under the lock through the resolver. A parent that moved since the lock-free merge then runs the conflict session with the lock held. In a batch of more than one, the same stale merge gets outcome `conflict` instead.
- **Evidence:**
  - `internal/landingshed/finalize.go:1089-1095` and `1404`: `mergeAndGate` and `mergeParent` call `resolver.Resolve` under the lock.
  - `internal/landingshed/finalize.go:682-689`: `freeLock` is called only for the fixer and the fix review.
- **Fix:** extend D8's "a merge under the lock that would need a conflict session frees the lock and runs the lock-free merge again first" to every under-lock merge D3, D6 and D8 run.

F-c — MEDIUM, design, `_lym/discussion/decision-record.md:103`
- **Summary:** D6 judges a member's owner, reconcile, land check and pull-request resolver only after the chain is gated, and leaves a failing member with no outcome. D3 judges only pause and tip at take time. So each later leader takes the same disqualified member again and pays a full rebuild and gate for it, until the member's own Finalize gets the lock. The rebuild bound caps this per hold, not per member.
- **Evidence:** `decision-record.md:62-63` (D3 skips) against `:105-111` (D6 checks after the gate, left out with no outcome).
- **Fix:** run D6's member reads at batch take in D3 as well, and keep the pre-move re-read; or mark a left-out member so later leaders skip it until its own Finalize acts.

F-d — MEDIUM, design, `_lym/discussion/decision-record.md:111`
- **Summary:** when a member of D7's passing prefix fails D6's pre-move checks, D6 says the leader "builds the chain again without it and gates again". It does not say whether the rebuild covers the passing prefix only or the whole batch, nor when the culprit gets `failed`. A whole-batch rebuild takes in the known culprit again and bisects it again; a prefix-only rebuild leaves the culprit's outcome timing undefined.
- **Evidence:** `decision-record.md:111-114` against `:135-136`.
- **Fix:** state that after a bisection the rebuild covers the passing prefix only, and that the culprit's `failed` is written when the culprit is found.

F-e — MEDIUM, design, `_lym/discussion/decision-record.md:74`
- **Summary:** D3 puts the leader's own entry first, and D4 gives any entry whose merge onto the chain is not clean outcome `conflict`. After D3's sync moves the parent past the leader's lock-free merge, the leader's own entry can conflict. The record does not say whether that leader still gates and lands the rest of its batch, or follows D8's `conflict` bullet, which "frees the lock" and "leads no batch".
- **Evidence:** `decision-record.md:62`, `:74` and `:159-160`.
- **Fix:** state in D4 or D8 that a leader whose own entry conflicts sets `conflict`, takes no batch and follows D8's `conflict` path.

F-f — LOW, consistency, `_lym/discussion/decision-record.md:228`
- **Summary:** the D3 criterion says "takes at most `landing.batch_max` entries in join order after its own", which reads as up to `batch_max` others plus the leader. D3:62 and D9:169 count the leader's own entry within `batch_max`.
- **Evidence:** `decision-record.md:62`, `:169` and `:228`.
- **Fix:** reword the criterion as "at most `landing.batch_max` entries, its own first, then in join order".

**Tool calls**
- grep of `VerifyDir` wiring in `internal/loomcli/landingdeps.go` and `verifytree.NewPaths`: to confirm the verified-tree record is per worktree, which my context did not show.
