# Support log: bolt-and-store

## Interview

The session ran autonomously; no operator turn was available.
Every batch below is a question the design raised and the pick made on it.

Batch 1, scope and problem.
Is the task right as posed, or is the ping-pong a symptom of task worktrees building their own lyx?
Pick: both; forward-only ordering (D1) fixes the two-production-binaries case the issue leaves open across machines, and channel gating (D2) closes the task-worktree-build case the issue names as the likely culprit.
Parts 3 and 4 were kept in scope as the board entry says the operator chose.

Batch 2, ordering source.
Where does "newer" come from: a content version counter, the build revision, or the registry fingerprint?
Pick: the build revision with git ancestry, because it needs no new versioning discipline and both binaries agree on it.

Batch 3, channel.
Is the production stamp needed once ancestry exists?
Pick: yes, for the task-branch build that is a descendant of main's recorded commit but not an ancestor of main's next deploy.

Batch 4, pull cost.
Can the pre-run afford a fetch on every command?
Pick: no; the fetch is gated on a due write, which keeps the common case free.
The reported incident was a lagging clone that committed seed output on top of a Bolt 1829 commits behind its upstream.

Batch 5, conflict resolution.
Regenerate the seed inside the push, or drop and let the next pre-run regenerate?
Pick: drop and regenerate at the next pre-run, so the push path carries no seeding logic and the forward rule is applied in one place.

Batch 6, drop scope.
Does the drop cover the hub-wide config commit?
Pick: no; the build stamp would then claim a reconcile the pulled copy undid, so the config gets pull-first and a stamp-absent retry instead.

Batch 7, in-flight signal.
Board run statuses or live reed sessions?
Pick: reed sessions, because a crashed run leaves a stale board status and would block gc for good.

Batch 8, loose objects.
Prune at landing, rate-limit the gc, or shorten the prune expiry at removal?
Pick: shorten the expiry in the existing housekeeping gc, under D8's quiet condition.

Batch 9, reporting.
Should `hubreconcile.Ensure` fail a start verb on a push failure the way `fabric reconcile` now fails?
Pick: no; a start verb must not die on a push blip, and the absent stamp makes the next start retry.

## Rejected alternatives

- Seeding only from the production binary, with no ancestry rule: two machines deploying different main commits against one Bolt would still flip.
- A per-subtree manifest file recording one revision for all stencils: a stencil untouched for months would be re-stamped on every pass, and a hand-edited file would share a revision with files it was not seeded with; per-file stamping in the banner keeps the existing classify table intact.
- A fetch on every pre-run: a network call before every lyx command, including the read-only ones agents run in loops.
- Pushing the seed commit at the pre-run instead of riding the next board push: the same network cost on every pass that writes, for a window the drop rule already closes.
- Regenerating the seed inside the push retry: the push path would need the registry, the mode and the ancestry read, duplicating the pre-run.
- Including the hub-wide config commit in the drop set: the build stamp would say the hub is reconciled while the pulled copy holds an older config.
- Commit time as the only ordering, with no ancestry read: exposed to clock skew even where the worktree holds both commits and ancestry is exact.
- Leaving a consumer hub's stencils unrefreshed once its worktree cannot establish ancestry: a regression from today, where a production binary refreshes an untouched copy.
- A drop that refuses whenever a non-seed commit is ahead: the board commit that triggers the push is nearly always ahead, so the drop would almost never fire.
- Pushing seed commits at once so they never share an ahead set with board commits: a push on every writing pass, and a failed push still leaves the seed beneath the next board commit.
- Board run status as the in-flight signal: persisted state that an abandoned run leaves behind.
- A scheduled or deferred gc after a skipped one: a new daemon duty for a case the next quiet removal covers.
- A rate limit of one gc per store per interval: bounds the repeat but leaves the loose count high, which is the reported cause.
- A prune at landing: landing runs inside a pair while other runs are in flight, the condition Part 3 wants gc kept out of.
- A hard ceiling on the loose count that forces a gc under live runs: reintroduces the repack under other runs Part 3 removes.

## Review rounds

_No rounds yet._

## Question ledger

- Auto-pick: forward-only by build ancestry, recorded per file in the stamp line (D1). Rationale in Interview batch 2.
- Auto-pick: commit time as D1's fallback where the worktree lacks either commit, covering consumer hubs and the specs pass (review round 1).
- Auto-pick: `lyx stencil sync` commits under `lyx: sync <subtree>` so D5 never drops an operator's forced refresh (D3, review round 1).
- Auto-pick: D5's primitive drops seed commits and replays the remaining ahead commits onto upstream, restoring the original tip on a replay conflict (review round 1).
- Auto-pick: an unpushed hub-wide config commit keeps the stamp absent and warns at each start verb until the operator acts (D4, review round 1).
- Auto-pick: a probe with no tmux server reports no sessions; any other probe failure skips gc with a Warn (D8, review round 1).
- Auto-pick: `lyx stencil sync` from a binary D2 does not let refresh writes no recorded revision, so a forced refresh never blocks a later production refresh (D1, review round 2).
- Auto-pick: D5's signature requires D3's label, so a pre-change seed or sync commit is never dropped and keeps today's push failure (D5, review round 2).
- Auto-pick: a conflicting hub-wide config commit clears only through the operator's `git pull --rebase` in the board dir; a board push clears only a transport failure (D4, review round 2).
- Auto-pick: a production channel stamped by the deploy tool, and an unstamped or modified build never refreshes (D2). Rationale in batch 3.
- Auto-pick: the seed commit label reuses `hubreconcile`'s revision label form (D3).
- Auto-pick: the fetch is gated on a due write; transport failure skips the pass with a warning (D4). Rationale in batch 4.
- Auto-pick: a gated drop primitive in `destroy.go` with a message-prefix-and-path signature, applied in Bolt's push path (D5). Rationale in batch 5.
- Auto-pick: the hub-wide config commit is outside the drop set; `hubreconcile` leaves the stamp absent on a push failure (D4, D5). Rationale in batch 6.
- Auto-pick: `fabric reconcile` exits non-zero on a failed hub-config or warp-binding push (D7).
- Auto-pick: in-flight means another pair's live reed session; the probe is injected (D8). Rationale in batch 7.
- Auto-pick: `gc.pruneExpire=1.day.ago` in the housekeeping gc; `PATTERN-hub-store-gc` is amended (D9). Rationale in batch 8.
- Auto-pick: `hubreconcile.Ensure` keeps its push failure non-fatal (batch 9).
- Open: whether `contracts/specs/refusal-spec.md` covers fabric refusals decides if D7 adds a spec row; the Plan producer checks.
- Open: the exact banner syntax for the recorded revision; left to the Plan producer with the note in the decision record.
- Open: whether the deploy tool's production route is reachable for a channel stamp without touching `update-plugins.sh`; the Plan producer checks `tools/deploy`.
- Environment: nothing outside the two discussion files was found broken during Step 1 and Step 2.
