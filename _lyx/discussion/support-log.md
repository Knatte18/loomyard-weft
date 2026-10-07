# fabric-pushes — support log

## Interview

Autonomous mode: no operator answered; each turn below is the agent's own question, the evidence read, and the pick.

1. Scope check against the board.
   The entry bundles #433 (retry and rollback), the prime-side push from run-pushes' PR-Gate, and the vocabulary exemption follow-up from `pattern-init`.
   Running in parallel: `batcher-v2` (touches `tools/tokencount`), `lifecycle-waits`, `burler-cost`; none plans fabric push or rollback work.
2. Where does the retry go?
   `Add` pushes with raw `gitexec.Run("push", "-u", "origin", …)` at steps 11 and 12; the anchored pushes already retry a non-fast-forward once and only warn on failure.
   Evidence: issue #433 (2026-10-07), where three simultaneous pair creations pushed to the same weft repo and GitHub answered one of them with a bare `(failed)`; a retry clears it.
   Pick: retry inside `Add` only, both pushes, bounded, classifying only a `! [remote rejected] <ref> -> <ref> (failed)` line whose reason is exactly `failed`, whatever other lines accompany it.
3. Why does the rollback leave the warp branch?
   `rollbackAdd` step 5 uses `ownedManagedBranch`, which can prove ownership only by a non-empty `branch_prefix` or a `-weft` suffix; under the default empty prefix it always refuses.
   Its doc also states that no origin branch is ever touched, because an unattended rollback should make no network-visible destruction.
   The hub had to delete the left-behind branch locally and on origin by hand before the resumed run went through.
   `probeWarpLeftover` returns no adopt tip both when origin has no such branch and when the pair is not live and origin's tip is at or behind `HEAD`; in the second case step 11 pushes an existing branch forward, so "not adopted" does not mean "created by this call".
   Pick: prove ownership by creation in this call (a minted proof, like `createdToken` for worktrees), delete locally always, on origin only when the pre-flight probe found no branch there and this call attempted step 11's push, leased to the commit it pushed.
   An exhausted step 11 is covered by the same rule: the lease refuses harmlessly when nothing landed and deletes the ref when it did.
   `Add` skips the probe under `SkipGit` or `SkipPush` while step 11 still pushes, so a skipped probe is not an absent answer: there the rollback deletes the local branch only.
4. How does the prime publish `main`?
   `PushPairAnchored` pushes the code side unconditionally; `sync` hands both paths to `CoalescePushBothAt`; `lyx config` runs `fabric sync` after every edit.
   `Fabric.Commit` also spawns a detached push of both sides whenever a commit lands, which `fabric commit`, `fabric sync` and every other `Commit` caller reach.
   `fab.Commit` commits the records side only, so in the prime no code-side commit is ever the call's own.
   `Fabric.Commit` already resolves a `Location` from its warp path, so it decides the prime itself, and a resolution failure there errors before any lock or commit.
   Pick: in the prime, every route that hands the code side to a push drops it, with a `code_push_skipped` report on `fabric push`.
5. What is in the exemption set today?
   `vocabExempt` has grown past the identifier set into env vars, envelope keys, a trace detail, refusal wording, a lock-dir name, file names, guard tokens and retired spellings.
   Pick: rename every name (D4); move or route the non-name spellings into the owner set (D5); delete the mechanism (D6).
6. Which neutral vocabulary?
   Fabric's prose and the `cmd/lyx` guard reasons already say "code side", "records side", "records worktree".
   Pick: warp → code, weft → records; env vars take the `FABRIC_` prefix since they cover both sides.

## Rejected alternatives

- Retrying every push failure in `Add`: a hook rejection or non-fast-forward is not transient and would only delay the rollback.
- Retrying `(failed)` in the anchored pushes as well: their failure is a warning and the next transition re-pushes, so nothing needs it.
- Keeping the rollback local-only and leaving the pushed warp branch on origin: the resumed run then refuses on the leftover, which is the #433 failure.
- Deleting the warp branch on origin without a lease: it could delete a push another actor made after step 11.
- Recognising the warp branch by name under an empty prefix: indistinguishable from an operator's own branch, the reason `ownedManagedBranch` refuses.
- Dropping the prime's code side only in `PushPairAnchored` and the `sync` verb's spawn: `Fabric.Commit`'s own spawn would still publish `main`.
- Deleting the origin warp branch whenever the local branch was forked: a pre-existing origin branch at or behind `HEAD` is also forked from, and step 11 only pushes it forward.
- Refusing `fabric push`/`sync` in the prime when `main` has unpushed commits: it would also break `lyx config` edits whenever the hub holds a local commit, which is the hub's normal state between commit and verified push.
- Keeping the hub from holding unpushed `main` commits: a process rule that the next mistake breaks; the code path stays unsafe.
- Identifying a task pair by its origin record instead of "not the prime": a pair without the record would lose its code-side push, regressing run-pushes.
- Keeping old env var names, envelope keys or `side=warp` as aliases: no versioning and no outside consumer.
- Moving `cmd/lyx/destructiveguard_test.go` and `uncontainedwrite_test.go` into `internal/fabricengine`: `cmd/lyx` is where every module-wide guard test lives.
- Putting the guard tests on `vocabScanAllowlist`: it moves the exemption rather than removing it.
- Renaming the `.weft` lock directory on disk: every live pair would need a migration for no behavioural gain.
- Keeping `vocabExempt` as an empty list: dead machinery.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Retry scope: which pushes retry a bare `(failed)`? — resolved, auto-pick: `Add`'s warp and weft pushes only (D1).
- Q2 Retry bound: how many attempts? — resolved, auto-pick: at most 4 attempts, exponential backoff with jitter, about 10 seconds total (D1).
- Q3 Rollback ownership proof — resolved, auto-pick: a proof minted at branch creation in `destroy.go` (D2).
- Q4 Rollback remote deletion — resolved, auto-pick: only a branch the pre-flight probe ran on and found absent on origin, and this call attempted to push, leased to the pushed commit (D2).
- Q5 Adopted warp branch on rollback — resolved, auto-pick: local copy deleted, origin untouched (D2).
- Q6 Prime push: gate, refuse or records-only? — resolved, auto-pick: records-only on every route, `Fabric.Commit`'s spawn included, with a `code_push_skipped` report on `fabric push` (D3).
- Q7 Prime predicate — resolved, auto-pick: `PrimeName(l) == l.WorktreeName`, decided in each route from the `Location` it holds or resolves; a resolution failure fails the call before any commit or push (D3).
- Q8 Neutral vocabulary — resolved, auto-pick: code/records, `FABRIC_` env prefix (D4).
- Q9 Compatibility for renamed env vars, keys, trace detail — resolved, auto-pick: hard cut (D4).
- Q10 Non-name spellings (file names, guard tokens, retired spellings, `.weft`) — resolved, auto-pick: rename, move into the owner set, or reach through a fabric helper (D5).
- Q11 Exemption mechanism — resolved, auto-pick: deleted (D6).
- Q12 Overlap with `batcher-v2`'s `tools/tokencount` changes — open, recorded as a risk; resolved at landing.
