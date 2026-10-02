# Support log — webster-recovery

## Interview

Autonomous session: no operator answered; every pick below is the Discussion producer's own, recorded in the Question ledger.

1. Task read from the board (`webster-recovery`, bug, tier 1) and issues #329, #330, #332 read on GitHub.
   Overlap checked: `verify-publish-hardening` owns what a card verifies and Master's re-run after rework; `run-parentage` and `operator-surface-fixes` do not touch webster.
2. #329, explored: `begunCards` scopes every begun batch out of the resolve pass, and `createTargetSet` is built only from pending cards, so an unlanded begun card's Create target vanishes for later `Uses:`.
   `validate` scopes by terminal batches only, which is why it disagreed with `begin-batch`.
   Picked: begun non-terminal cards' Create/Rename-New targets stay forthcoming (D1), one scoping rule for all three sites (D2).
   Challenged the board's "only landed batches count as built": attributing commits to a batch needs a git walk that breaks when `begin-batch` runs out of order or a batch lands partly, and re-validating a re-begun batch's unchanged cards adds nothing the begun-card hash guard does not already ensure.
3. #330, explored: `BindHandles`, drift repair and `CanonicalizeHandles` all write through plan-wide `RewriteRefs`, so a later batch's rewrite reaches begun cards, and only `PlanFingerprint`/`PlanFileHashes` are restamped.
   Picked: the same restamp moves begun `CardHashes` for cards whose begun hash equals the adopted pre-rewrite file hash (D3).
4. #332a, explored: the implementer body runs only `go build ./...` and the card's package tests, plus its `verify:`; the Seen failures were repo-wide enforcement tests the forks saw and reported.
   Picked: a prompt rule that a failure the card caused is the card's gate failure under the self-fix cap (D4); widening what the fork runs is left to `verify-publish-hardening`.
5. #332b, explored: triage and bisect run in `run`'s exit stage after Master has ended, and the stage already phases its long work outside the lease.
   Picked: one cold fix strand at the recovery role, Go-checked commits, Go re-triage decides (D5), recorded and reclaimable (D6).
6. Testing: one regression test per Seen scenario plus the guard-holds negatives, listed under the decision record's acceptance criteria.

## Rejected alternatives

- #329, count a begun batch as built only when commits landed since its `StartSHA` (the board's direction): needs commit attribution; misattributes another batch's commits when batches begin out of order; still drops unlanded targets when a multi-card batch lands partly.
- #329, count every card's Create target forthcoming, terminal ones included: a later `Uses:` of a target a completed card should have created would stop being checked.
- #330, `rebaseline` compares begun cards modulo handle canonicalization: needs a canonicalization-aware diff and still misses binding and drift-repair rewrites, which are not canonicalization.
- #332a, make the fork run `go test ./...` or the repo scans: decides what a card verifies, which `verify-publish-hardening` owns.
- #332b, move triage and bisect into a Master-called verb so Master forks the fix in-session: the verify reruns take minutes to tens of minutes, past what one Bash call can hold before Claude Code backgrounds it, and the restructure is far larger.
- #332b, reopen the offending batch and route it through `recover-batch`: that batch's `StartSHA`, delta, done-checks and suspect paths describe its own position, with later batches already landed on top.
- #332b, route the regression to Webster-Review: a review row judges, it does not fix, and the run is already blocked by then.
- #332b, a configurable attempt cap: no requirement for more than one attempt; one attempt is the bound.
- #332b, reset the branch to the pre-fix head when Go rejects the strand's commits: unsafe when HEAD moved by something other than the strand, and it discards what the operator needs to inspect.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): how should #329 decide a re-begun batch's targets? → D1, forthcoming targets; rejects the board's landed-commits direction (see Rejected alternatives).
- Q2 (resolved, auto-pick): should `lyx webster validate` change scoping? → yes, D2.
- Q3 (resolved, auto-pick): #330 hash update at rewrite or canonicalization-aware `rebaseline`? → at rewrite, D3.
- Q4 (resolved, auto-pick): how does a fork tell its own breakage from someone else's? → the test exercises what the card changed, or a scan flags a file the card wrote; D4.
- Q5 (resolved, auto-pick): who fixes an integration regression? → a cold strand at run exit, D5.
- Q6 (resolved, auto-pick): how many fix attempts? → one per run, recorded when spawned, no config key.
- Q7 (resolved, auto-pick): attempt a fix when bisect did not localize a card? → yes, card hint omitted.
- Q8 (resolved, auto-pick): model role and timeout for the fix strand? → the recovery role and `recovery_timeout_min`.
- Q9 (resolved, auto-pick): repair stale `CardHashes` of runs begun before deploy? → no; listed under Open risks.
- Q10 (resolved, review round 1): what happens to strand commits Go rejects? → they stay on the branch, named in summary.md and the escalation's way-forward; D5.
