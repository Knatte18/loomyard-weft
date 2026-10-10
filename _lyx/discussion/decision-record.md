# Decision record: bolt-and-store

## Goal

lyx's own git writes to Bolt and to the shared hub object stores stay correct and quiet.
Stencil and spec seeding moves Bolt's copies forward only, never back to an older binary's default, and every seed commit names the binary that made it.
Seeding and the hub-wide config write start from an up-to-date Bolt, a seed commit that conflicts at push time is dropped and regenerated instead of left behind, and `lyx fabric reconcile` reports a failed push as a failure.
The hub stores' gc at pair removal does not repack under other runs, and prunes loose objects so it stops repacking at every teardown.

## Scope

In:

- `cmd/lyx`'s root pre-run seed pass, `internal/stencilstore`'s refresh rule and stamp, `internal/stencilcli`'s sync commit message, `internal/fabricengine`'s seed commit and Bolt push path, `internal/hubreconcile`'s hub-wide config write, `internal/fabriccli`'s `reconcile` envelope and exit code, `internal/fabricengine/housekeeping.go`, `internal/buildinfo`'s channel and the deploy tool's stamping of it, `internal/buildvcs`'s commit time.
- The `doc.go` of every package above, `docs/overview.md` where the stencil row describes the refresh rule, `PATTERN.md` and `pattern/PATTERN-hub-store-gc.md`.

Out:

- Board sync commits and their push path beyond the one conflict rule in D5; the board's own write locks are unchanged.
- The per-worktree config commits of `hubreconcile`'s walk; only its hub-wide board commit changes.
- The warp-binding backfill in `fabric reconcile`, except that its push failure now fails the verb (D7).
- Any versioning of stencils beyond the build revision and commit time, any operator configuration of gc, and any new CLI verb.
- A gc or prune at landing: loose-object growth is handled in housekeeping at removal (D9).

## Decisions

### D1. A stencil refresh is forward only, by build ancestry

Decision: the stamp line a seeded file carries in its leading banner records, beside the body hash, the VCS revision and commit time of the binary that wrote it; `buildvcs` reports the commit time beside the revision.
`reconcileOne`'s refresh row, which overwrites an untouched board copy whose body differs from the shipped default, runs only when the running binary is newer than the recorded writer, or when the file carries no recorded revision.
Newer is decided by ancestry when the code worktree the command runs in holds both commits: `gitrepo`'s object-presence read checks that both exist, and its reachability read, `git merge-base --is-ancestor` through `gitexec`, answers whether the recorded revision is an ancestor of the running one.
When the worktree lacks either commit, as in a hub whose code repository is not loomyard or a task worktree behind the installed binary, newer means the running binary's commit time is strictly later than the recorded one.
A file the applicable rule does not find older is left alone with one Info line naming the stencil, both revisions and `lyx stencil sync` as the override.
The specs pass gets the same answer through its `Source`, whose source directory stays empty.
`ForceRefresh`, behind `lyx stencil sync`, ignores the ordering rule as it ignores the dev rule today.
It writes the running binary's revision and commit time only when D2 lets that binary refresh: production channel, a revision and an unmodified tree.
From any other binary it writes the body with no recorded revision, so the next production binary refreshes the copy as it does any unstamped one, and a forced refresh from a dev, unstamped, task-branch or modified build never blocks a later production refresh.
`BodyHash`, `Classify` and the edit-detection table are unchanged: the revision and time live in the banner, which the hash never covers.

Bound: only a production binary refreshes (D2), and it is built from a clean, pushed main, so a later deploy's commit descends from an earlier one's and its commit time is later unless the committing machine's clock was skewed.
A skewed time can cause a refresh in the wrong direction under the commit-time rule, never a refresh by a dev, unstamped, modified or task-branch build.

Rationale: a body hash says that two versions differ and nothing about which is newer, which is the whole ping-pong.
Commit ancestry is the one ordering both binaries agree on, and it is exact wherever the history is present; commit time stands in where it is not, sound for production builds because each deploy's commit descends from the previous one's.
A file with no recorded revision is every stencil Bolt holds today, so the first production binary after this lands refreshes them once and stamps them.

### D2. Only a clean, stamped, production-channel binary refreshes

Decision: `buildinfo` gains a production channel beside `dev`, stamped by the deploy tool on its production route, the one `update-plugins.sh` runs.
`stencilstore.Mode` gains a third value for an unstamped binary, and the pre-run chooses the mode from the channel: production refreshes under D1, dev warns as today, and unstamped seeds absent files and warns like dev, naming `lyx stencil sync` and `update-plugins.sh` as the way forward.
A binary whose VCS stamp reports a modified tree, or carries no revision, never refreshes an untouched file whatever its channel; it seeds absent files only and warns once per stencil.
The zero value of `Mode` becomes the unstamped mode, so an unset mode never refreshes.

Rationale: forward ancestry alone still lets a clean build from a task branch seed Bolt with a commit main does not descend from, after which the production binary can never move it back until the task lands.
The deploy script is the one production route the repo already names, so the stamp costs nothing to place and tests that need a refreshing binary build one with the production ldflag through `lyxbin`.
A modified build has no commit that names its content, so no ordering exists for it.

### D3. Every seed commit names its binary

Decision: a pre-run seed commit's message is `lyx: seed <subtree>` followed by a parenthesised label with the short revision and its `-modified` suffix in `hubreconcile`'s existing label form, the channel, and the executable's path.
`lyx stencil sync` commits under its own prefix, `lyx: sync <subtree>`, with the same label, so D5's signature never matches an operator's forced refresh.
The label is built in one place and shared with `hubreconcile`'s hub-wide commit message.

Rationale: the next flip, if any, is traceable from `git log` on Bolt alone.

### D4. Seeding and the hub-wide config write pull Bolt first, and only when they have something to write

Decision: the pre-run classifies every registry name first and reaches for the network only when at least one write is due.
Then, under the board write lock, it fetches Bolt's upstream, drops any seed commits ahead of upstream through D5's primitive, fast-forwards Bolt when it is only behind, classifies again against the pulled copy and writes what is still due, and commits as today.
The commit rides the next board push as before.
A fetch that fails as a transport failure skips the writes for this process with one Warn naming the remote and the next command as the way forward.
A Bolt that still diverges from its upstream, because it holds non-seed commits over a moved upstream, or because D5's replay conflicted, skips the writes with one Warn naming the board dir, `git pull --rebase` in it and the existing board push as the way forward.
`hubreconcile`'s hub-wide write and `fabric reconcile`'s hub-config heal run the same fetch, drop and fast-forward before writing, through one `fabricengine` helper on `Bolt`.
Their commits are pushed at once as today; `hubreconcile` leaves the build stamp absent when that push fails, so the next start verb reruns the walk.
A hub-wide config commit whose push failed stays ahead of a moved upstream, so the rerun walk meets the divergence rule above: the stamp stays absent, every start verb repeats the Warn, and pre-run seeding with a write due skips.
When the push failed in transport, the next board push lands the commit and clears the state.
When it failed on a rebase conflict, a board push meets the same conflict, since D5 never drops a hub-wide commit, so only the operator resolving `git pull --rebase` in the board dir clears it.

Rationale: a no-op pass is the common case on every lyx invocation and must stay free of network, so the pull is gated on a due write.
Comparing against upstream instead of a stale local copy is what stops a clone behind its upstream from committing on top of a stale copy.
The stamp-absent retry reuses the mid-merge precedent `hubreconcile` already documents.

### D5. A seed commit that conflicts at push is dropped and regenerated

Decision: `destroy.go` gains one gated primitive that removes seed commits from Bolt's ahead set: it resets Bolt to its upstream through `ResetKeep` and replays the remaining ahead commits onto it in order.
A seed commit is one whose message is the pre-run's seed prefix, `lyx: seed <subtree>`, followed by D3's parenthesised label, and every path it touches lies under the stencils or specs subtree; a `lyx: sync` commit and a hub-wide config commit never match.
A commit made by a pre-change binary, whose pre-run and `lyx stencil sync` both commit under a bare `lyx: seed <subtree>`, carries no label and never matches, so a conflict on it fails the push as today.
Both conditions are declared once in `fabricengine`, beside the subtree helpers.
The primitive runs with containment, ownership, dirtiness and never force, in the gate's order, and appends to the mutation record.
When a replayed commit conflicts, the primitive aborts the replay, restores Bolt to the tip it started from and refuses.
Bolt's push path, on a `pull --rebase` that fails, aborts the rebase as today, then, when the ahead set contains seed commits, runs the primitive and retries the rebase and push once.
The dropped content is regenerated by the next pre-run of a binary D1 and D2 allow to refresh.

Bound: the primitive discards only commits holding files a seeding pass wrote: absent files seeded from a shipped default, untouched copies refreshed to one, restamps of a body equal to a shipped default and the seeded `.gitattributes`, never a hand edit or an operator's forced refresh.
Until a refreshing binary runs, Bolt holds the pulled upstream copy of those files.
Every non-seed commit is replayed onto upstream or, on a conflict, kept at its original tip, leaving the push failure and its way forward as today.

Rationale: a seed is derived content, so the right resolution of a conflict is to discard the local derivation and let the forward rule rederive it against the pulled copy.
A seed commit reaches the push path beneath the board commit that triggered the push, so the drop must keep that commit rather than refuse because of it.
Resetting Bolt is destruction and so belongs behind the chokepoint, with a signature narrow enough that no board or operator commit can match it.

### D6. The way forward in a seed or push failure never names a denied command

Decision: every new Warn and refusal names `lyx stencil sync`, `update-plugins.sh`, the existing board push verb, `git pull --rebase` in the board dir, or a re-run; never a reset or force.

Rationale: `PATTERN-no-denied-recovery`.

### D7. `lyx fabric reconcile` fails when the hub config or the warp binding did not push

Decision: a commit or push failure of the hub-wide config or of the warp-binding record exits non-zero with the envelope kept, the detail fields as today, and an error naming the first failure and the way forward.
The verb's long help drops the sentence saying a push failure leaves the exit code unchanged.
The refusal follows `PATTERN-refusal-way-forward`'s shape and gets a spec row if `contracts/specs/refusal-spec.md` lists fabric refusals.

Rationale: `ok: true` beside a detail that says the push failed is what let the stale clone repeat the failure unnoticed.

### D8. Store gc at removal is skipped while another pair's run is in flight

Decision: `housekeepStores` takes an in-flight probe and skips the gc, keeping the key writes, when the probe reports another run; the skip is logged at Info with the sessions found.
The probe lists reed's live sessions on the hub's tmux socket and reports in flight when any belongs to a pair other than the one being removed; the prime's own sessions, the orch among them, never count.
A listing that finds no tmux server on the socket reports no sessions.
Any other listing error counts as in flight, and that skip logs at Warn with the error instead of Info.
Both callers of removal, `fabric remove` and the batten teardown, pass the real probe.
There is no deferred or scheduled gc: the next removal that finds no run runs it.

Bound: on a hub where every removal overlaps another run, or where the probe fails at every removal, the stores are not gc'd by lyx until a quiet removal; the failing case shows as a Warn at each removal, and the operator's manual `git gc` stays the override.

Rationale: a live reed session is the one hub-wide, non-persisted sign of a running agent or driver, so it cannot go stale the way a board status can after a crash.
Injecting the probe keeps the skip testable at the integration tier.

### D9. Housekeeping prunes loose objects on a one-day expiry

Decision: the foreground `gc --auto` in `housekeepStores` passes `gc.pruneExpire=1.day.ago` beside its existing settings, so unreachable loose objects older than a day are pruned by the same gc and the loose count falls.
This newly lets gc delete unreachable loose objects between one day and two weeks old, which git's default expiry kept; the one-day expiry alone bounds it, since D8 does not count the prime's sessions and an operator's git command is outside any probe.
This changes `PATTERN-hub-store-gc`'s line that gc deletes only objects past git's own default expiry; the entry and its background are updated in the same commit to name the one-day expiry and the in-flight skip.
Reflog expiry and `gc.worktreePruneExpire=never` are unchanged.

Rationale: git's two-week default keeps every young unreachable object loose, so `gc --auto` finds the count over its threshold at every removal and repacks without lowering it.
A day covers any lyx or operator command still in progress whose new objects are not yet referenced.

## Constraints

- `PATTERN-hub-store-gc`: housekeeping stays the sole gc site and the sole writer of the keys; D9 changes the expiry line and D8 adds the skip, both recorded in the entry.
- `PATTERN-fabric-destruction-chokepoint`: D5's reset, replay and restore live in `destroy.go` with the four checks in order.
- `PATTERN-mutation-record`: the drop primitive and the pull-first helper append only after an observed change.
- `PATTERN-gitrepo-client-boundary` and `PATTERN-gitexec-checked-call`: fetch, pull, reset, replay, push and D1's reachability read through `gitexec`; D1's object-presence check through `readGoGit`.
- `PATTERN-stencil-ownership`: seeding stays once per pre-run, never lazy; `stencilstore` runs no git and is told the ordering answer through its `Source`, extended to answer the D1 question.
- `PATTERN-lyxdirs-single-declarer`: the D5 subtree containment reuses the existing subtree helpers.
- `PATTERN-leaf-packages`: `buildinfo` stays import-free; `buildvcs` stays stdlib-only while gaining the commit time.
- `PATTERN-refusal-way-forward` and `PATTERN-no-denied-recovery`: D6 and D7.
- `PATTERN-test-speed`, `PATTERN-test-isolation`, `PATTERN-test-economy`, `PATTERN-hubforge-fixtures`: Bolt tests build on `hubforge`; the housekeeping skip tests at the integration tier through the injected probe.
- `PATTERN-documentation-lifecycle` and `PATTERN-comment-line-breaks`: docs in the same commit, semantic line breaks.
- Test budget: a package whose new top-level tests pass its row in `cmd/lyx/testdata/test-budget.yaml` lists that file in the card.

## Auto-mode assumptions

- The deploy tool's production route can stamp a channel the same way its dev route does, and `update-plugins.sh` reaches it.
- Go's default VCS stamping gives `lyxbin` test builds a revision and commit time, so a test that needs a refreshing binary adds only the production channel ldflag.
- Reed's session listing can tell a pair's session from the prime's by name, through `internal/agentname`'s forms.
- `ResetKeep` suffices for the drop: a seed commit touches only seed subtrees, so uncommitted board writes never collide with it.
- `fabric reconcile`'s hub-config heal is rare enough that an unconditional fetch there is acceptable.

## Open risks

- A pre-change binary still in use strips the recorded revision when it overwrites a stencil, and the new binary refreshes again; the flip ends when no pre-change production binary runs against the hub.
- Where D1 falls back to commit time, a production build committed on a machine with a skewed clock can refresh a stencil backwards once.
- A hub-wide config commit whose push conflicts is not dropped by D5, and a board push meets the same conflict; it holds the hub-wide stamp absent and blocks due pre-run writes until the operator resolves `git pull --rebase` in the board dir, named in the Warn.
- A seed or sync commit made by a pre-change binary carries no D3 label, so D5 never drops it, and a push it conflicts fails as today until the operator resolves `git pull --rebase` in the board dir.
- A hub that is never quiet at a removal, or whose probe fails at every removal, is never gc'd by lyx.
- Pruning on a one-day expiry removes an unreachable object a manual operation older than a day still meant to reference; git's own default is two weeks.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was addressed by the round's fixer, and no fresh reviewer has seen the fix.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - forced-refresh-nonprod-binary-freezes-copy (class design, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 4: converged; each finding below was addressed by the round's fixer, and no fresh reviewer has seen the fix.
  Review: _lyx/reviews/plan/round-4-review.md; fixer report: _lyx/reviews/plan/round-4-fixer-report.md.
  - drop-not-serialised-against-push-rebase (class design, severity MEDIUM, open in rounds 4)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- A production binary whose revision descends from the recorded one refreshes an untouched, differing stencil and writes its revision and commit time; one whose revision does not leaves the file and logs the Info line.
- In a worktree lacking either commit, as in a hub over another repository, a production binary refreshes only when its commit time is later than the recorded one, and otherwise leaves the file and logs the Info line.
- The specs subtree follows the same ordering rule as the stencils subtree.
- A stencil with no recorded revision is refreshed once by a production binary.
- A dev, unstamped or modified-tree binary seeds absent files, never refreshes an untouched file, and warns naming the way forward; a `Reconcile` with the zero `Mode` never refreshes an untouched file.
- `lyx stencil sync` refreshes regardless of channel, ordering and modified state, and commits under `lyx: sync <subtree>`.
- `lyx stencil sync` from a production, clean binary records its revision and commit time; from a dev, unstamped or modified binary it records none, and the next production binary's pre-run refreshes that copy.
- A seed commit's message carries the short revision, the modified suffix when set, the channel and the executable path.
- A pre-run with nothing to write runs no fetch.
- A pre-run with a write due on a Bolt behind its upstream fast-forwards first and commits on top of the pulled copy; on a Bolt holding seed commits ahead it drops them first; on a Bolt holding non-seed commits over a moved upstream it writes nothing and warns.
- A fetch transport failure writes nothing and warns.
- `hubreconcile` and `fabric reconcile`'s hub-config heal fetch, drop seed commits and fast-forward Bolt before the hub-wide write.
- `hubreconcile` leaves the stamp absent when the push fails, and the next start verb, meeting the unpushed commit over a moved upstream, writes nothing and warns naming `git pull --rebase` in the board dir.
- Bolt's push, rejected with seed and board commits ahead of a moved upstream, drops the seed commits, replays the board commits onto upstream and lands them; when a replayed commit conflicts, Bolt is restored to its original tip and the failure is reported as today.
- The drop primitive refuses to drop a commit whose message lacks the seed prefix or D3's label, a `lyx: sync` commit, a hub-wide config commit and a commit touching a path outside the seed subtrees, and records its mutations.
- `lyx fabric reconcile` exits non-zero, with the envelope and detail fields intact, when the hub config or warp binding commit or push fails.
- Removal with another pair's session live writes the keys and runs no gc; with only the prime's or the removed pair's sessions, or no tmux server, it runs the gc; with a failing probe it runs no gc and logs a Warn.
- Housekeeping's gc prunes an unreachable loose object older than a day and keeps one younger.
- `PATTERN-hub-store-gc`'s background names the skip and the expiry; the `doc.go` files of the touched packages and the stencil row of `docs/overview.md` describe the new rules.

## Notes for the plan writer

- The stamp line is `<!-- lyx-stencil: sha256=<hex> -->` today; `ApplyStamp` replaces the hex in place and keeps the rest of the line, so `build=<revision>` and `time=<commit time>` keys after the hex are one option.
- `fabricengine.StencilSource` wraps `HeadContains` once per `Source`; that read answers object presence and HEAD reachability through go-git, while D1's two-revision reachability read is `gitrepo`'s `IsAncestor`, which errors rather than answering false when a commit is missing, hence D1's presence check first.
- Go's `debug.BuildInfo` carries `vcs.time` beside `vcs.revision`; `buildvcs` reads only the revision and modified flag today.
- `hubreconcile`'s `revisionLabel` is the label form D3 reuses.
- `CommitSeededStencils` deliberately uses the board write lock and not `Bolt.Sync`; the pull-first helper keeps that choice and pushes nothing.
- The push path's rebase-retry and abort live in `gitrepo`'s push file; the drop belongs in `fabricengine`, so the retry loop around the drop sits in `pushWeftAt`'s layer, not in `gitrepo`.
- `housekeepStores` is called from `Topology.Remove` after a successful teardown; the probe is a parameter of the housekeeping call so `Remove`'s signature can carry it from both callers.
