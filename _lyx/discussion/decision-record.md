# Decision record: bolt-and-store

## Goal

lyx's own git writes to Bolt and to the shared hub object stores stay correct and quiet.
Stencil and spec seeding moves Bolt's copies forward only, never back to an older binary's default, and every seed commit names the binary that made it.
Seeding and the hub-wide config write start from an up-to-date Bolt, a seed commit that conflicts at push time is dropped and regenerated instead of left behind, and `lyx fabric reconcile` reports a failed push as a failure.
The hub stores' gc at pair removal does not repack under other runs, and prunes loose objects so it stops repacking at every teardown.

## Scope

In:

- `cmd/lyx`'s root pre-run seed pass, `internal/stencilstore`'s refresh rule and stamp, `internal/fabricengine`'s seed commit and Bolt push path, `internal/hubreconcile`'s hub-wide config write, `internal/fabriccli`'s `reconcile` envelope and exit code, `internal/fabricengine/housekeeping.go`, `internal/buildinfo`'s channel and the deploy tool's stamping of it.
- The `doc.go` of every package above, `docs/overview.md` where the stencil row describes the refresh rule, `PATTERN.md` and `pattern/PATTERN-hub-store-gc.md`.

Out:

- Board sync commits and their push path beyond the one conflict rule in D5; the board's own write locks are unchanged.
- The per-worktree config commits of `hubreconcile`'s walk; only its hub-wide board commit changes.
- The warp-binding backfill in `fabric reconcile`, except that its push failure now fails the verb (D7).
- Any versioning of stencils beyond the build revision, any operator configuration of gc, and any new CLI verb.
- A gc or prune at landing: Part 4 is solved inside housekeeping at removal (D9).

## Decisions

### D1. A stencil refresh is forward only, by build ancestry

Decision: the stamp line a seeded file carries in its leading banner records, beside the body hash, the VCS revision of the binary that wrote it.
`reconcileOne`'s refresh row, which overwrites an untouched board copy whose body differs from the shipped default, runs only when the running binary's revision descends from the recorded one, read through `fabricengine`'s existing in-process ancestry read against the code worktree the command runs in, or when the file carries no recorded revision.
A recorded revision that is not an ancestor of the running binary's, or an ancestry the worktree cannot establish because it lacks either commit, leaves the file alone and logs one Info line naming the stencil, both revisions and `lyx stencil sync` as the override.
`ForceRefresh`, behind `lyx stencil sync`, ignores the ancestry rule as it ignores the dev rule today, and writes the running binary's revision.
`BodyHash`, `Classify` and the edit-detection table are unchanged: the revision lives in the banner, which the hash never covers.

Rationale: a body hash says that two versions differ and nothing about which is newer, which is the whole ping-pong.
Commit ancestry is the one ordering both binaries agree on, and the code worktree every lyx command already runs in holds the history to answer it.
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

Decision: a seed commit's message is `lyx: seed <subtree>` followed by a parenthesised label with the short revision and its `-modified` suffix in `hubreconcile`'s existing label form, the channel, and the executable's path.
The label is built in one place and shared with `hubreconcile`'s hub-wide commit message.

Rationale: the next flip, if any, is traceable from `git log` on Bolt alone.

### D4. Seeding and the hub-wide config write pull Bolt first, and only when they have something to write

Decision: the pre-run classifies every registry name first and reaches for the network only when at least one write is due.
Then, under the board write lock, it fetches Bolt's upstream, drops any local regenerable commits ahead of upstream through D5's primitive, fast-forwards Bolt, classifies again against the pulled copy and writes what is still due, and commits as today.
The commit rides the next board push as before.
A fetch that fails as a transport failure skips the writes for this process with one Warn naming the remote and the next command as the way forward.
A fast-forward that fails because Bolt holds non-regenerable commits ahead of a moved upstream skips the writes with one Warn naming the board dir and the existing board push as the way forward.
`hubreconcile`'s hub-wide write and `fabric reconcile`'s hub-config heal run the same fetch, drop and fast-forward before writing, through one `fabricengine` helper on `Bolt`.
Their commits are pushed at once as today; `hubreconcile` leaves the build stamp absent when that push fails, so the next start verb reruns the walk.

Rationale: a no-op pass is the common case on every lyx invocation and must stay free of network, so the pull is gated on a due write.
Comparing against upstream instead of a stale local copy is what stops a lagging clone from committing on top of 1829 missing commits.
The stamp-absent retry reuses the mid-merge precedent `hubreconcile` already documents.

### D5. A seed commit that conflicts at push is dropped and regenerated

Decision: `destroy.go` gains one gated primitive that resets Bolt to its upstream through `ResetKeep`, allowed only when every commit being dropped is a lyx seed commit: its message starts with the seed prefix and every path it touches lies under the stencils or specs subtree.
Both conditions are declared once in `fabricengine`, beside the subtree helpers.
The primitive runs with containment, ownership, dirtiness and never force, in the gate's order, and appends to the mutation record.
Bolt's push path, on a `pull --rebase` that fails, aborts the rebase as today, then, when the ahead set contains seed commits, drops them through the primitive and retries the rebase and push once.
A hub-wide config commit is never dropped: D4's pull-first and stamp retry cover it.
The dropped content is regenerated by the next command's pre-run under D1 and D2.

Bound: the primitive discards only commits a running binary regenerates on its next pre-run, and refuses the drop as a whole when any ahead commit fails the signature, leaving the push failure and its way forward as today.

Rationale: a seed is derived content, so the right resolution of a conflict is to discard the local derivation and let the forward rule rederive it against the pulled copy.
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
Both callers of removal, `fabric remove` and the batten teardown, pass the real probe; a probe that fails counts as in flight.
There is no deferred or scheduled gc: the next removal that finds no run runs it.

Bound: on a hub where every removal overlaps another run the stores are not gc'd by lyx until a quiet removal, and the operator's manual `git gc` stays the override.

Rationale: a live reed session is the one hub-wide, non-persisted sign of a running agent or driver, so it cannot go stale the way a board status can after a crash.
Injecting the probe keeps the skip testable at the integration tier.

### D9. Housekeeping prunes loose objects on a one-day expiry

Decision: the foreground `gc --auto` in `housekeepStores` passes `gc.pruneExpire=1.day.ago` beside its existing settings, so unreachable loose objects older than a day are pruned by the same gc and the loose count falls.
This changes `PATTERN-hub-store-gc`'s line that gc deletes only objects past git's own default expiry; the entry and its background are updated in the same commit to name the one-day expiry and the in-flight skip.
Reflog expiry and `gc.worktreePruneExpire=never` are unchanged.

Rationale: git's two-week default keeps every young unreachable object loose, so `gc --auto` finds the count over its threshold at every removal and repacks without lowering it.
With D8 no lyx run writes the store during the gc, and a day covers any operator command still in progress.

## Constraints

- `PATTERN-hub-store-gc`: housekeeping stays the sole gc site and the sole writer of the keys; D9 changes the expiry line and D8 adds the skip, both recorded in the entry.
- `PATTERN-fabric-destruction-chokepoint`: D5's reset lives in `destroy.go` with the four checks in order.
- `PATTERN-mutation-record`: the drop primitive and the pull-first helper append only after an observed change.
- `PATTERN-gitrepo-client-boundary` and `PATTERN-gitexec-checked-call`: fetch, pull, reset and push through `gitexec`; the ancestry read through `readGoGit`.
- `PATTERN-stencil-ownership`: seeding stays once per pre-run, never lazy; `stencilstore` runs no git and is told ancestry through its `Source`, extended to answer the D1 question.
- `PATTERN-lyxdirs-single-declarer`: the D5 subtree containment reuses the existing subtree helpers.
- `PATTERN-leaf-packages`: `buildinfo` stays import-free; `buildvcs` stays stdlib-only.
- `PATTERN-refusal-way-forward` and `PATTERN-no-denied-recovery`: D6 and D7.
- `PATTERN-test-speed`, `PATTERN-test-isolation`, `PATTERN-test-economy`, `PATTERN-hubforge-fixtures`: Bolt tests build on `hubforge`; the housekeeping skip tests at the integration tier through the injected probe.
- `PATTERN-documentation-lifecycle` and `PATTERN-comment-line-breaks`: docs in the same commit, semantic line breaks.
- Test budget: a package whose new top-level tests pass its row in `cmd/lyx/testdata/test-budget.yaml` lists that file in the card.

## Auto-mode assumptions

- The deploy tool's production route can stamp a channel the same way its dev route does, and `update-plugins.sh` reaches it.
- Go's default VCS stamping gives `lyxbin` test builds a revision, so a test that needs a refreshing binary adds only the production channel ldflag.
- Reed's session listing can tell a pair's session from the prime's by name, through `internal/agentname`'s forms.
- `ResetKeep` suffices for the drop: a seed commit touches only seed subtrees, so uncommitted board writes never collide with it.
- `fabric reconcile`'s hub-config heal is rare enough that an unconditional fetch there is acceptable.

## Open risks

- A pre-change binary still in use strips the recorded revision when it overwrites a stencil, and the new binary refreshes again; the flip ends when no pre-change production binary runs against the hub.
- A task worktree whose HEAD lacks the installed binary's commit cannot establish ancestry, so that worktree's commands never refresh; the next command in the prime does.
- A hub-wide config commit whose push conflicts in the race between D4's pull and its push is not dropped by D5 and needs `git pull --rebase` in the board dir by hand, named in the failure.
- A hub that is never quiet at a removal is never gc'd by lyx.
- Pruning on a one-day expiry removes an object a manual operation older than a day still meant to reference; git's own default is two weeks.

## Acceptance criteria

- A production binary whose revision descends from the recorded one refreshes an untouched, differing stencil and writes its revision; one whose revision does not, or whose ancestry the worktree cannot establish, leaves the file and logs the Info line.
- A stencil with no recorded revision is refreshed once by a production binary.
- A dev, unstamped or modified-tree binary seeds absent files, never refreshes an untouched file, and warns naming the way forward.
- `lyx stencil sync` refreshes regardless of channel, ancestry and modified state.
- A seed commit's message carries the short revision, the modified suffix when set, the channel and the executable path.
- A pre-run with nothing to write runs no fetch.
- A pre-run with a write due on a Bolt behind its upstream fast-forwards first and commits on top of the pulled copy; on a Bolt holding only regenerable ahead commits it drops them first; on a Bolt holding other ahead commits over a moved upstream it writes nothing and warns.
- A fetch transport failure writes nothing and warns.
- Bolt's push, rejected with seed commits ahead of a moved upstream, drops them and lands the remaining commits; with a non-seed commit in the ahead set the drop is refused and the failure is reported as today.
- The drop primitive refuses a commit whose message lacks the prefix or that touches a path outside the seed subtrees, and records its mutations.
- `hubreconcile` fetches and fast-forwards Bolt before the hub-wide write, and leaves the stamp absent when the push fails.
- `lyx fabric reconcile` exits non-zero, with the envelope and detail fields intact, when the hub config or warp binding commit or push fails.
- Removal with another pair's session live writes the keys and runs no gc; with only the prime's or the removed pair's sessions it runs the gc.
- Housekeeping's gc prunes an unreachable loose object older than a day and keeps one younger.
- `PATTERN-hub-store-gc`'s background names the skip and the expiry; the `doc.go` files of the touched packages and the stencil row of `docs/overview.md` describe the new rules.

## Notes for the plan writer

- The stamp line is `<!-- lyx-stencil: sha256=<hex> -->` today; `ApplyStamp` replaces the hex in place and keeps the rest of the line, so a `build=<revision>` key after the hex is one option.
- `fabricengine.StencilSource` already wraps `HeadContains` once per `Source`; D1 needs an is-ancestor read of two revisions, which `gitrepo` exposes.
- `hubreconcile`'s `revisionLabel` is the label form D3 reuses.
- `CommitSeededStencils` deliberately uses the board write lock and not `Bolt.Sync`; the pull-first helper keeps that choice and pushes nothing.
- The push path's rebase-retry and abort live in `gitrepo`'s push file; the drop belongs in `fabricengine`, so the retry loop around the drop sits in `pushWeftAt`'s layer, not in `gitrepo`.
- `housekeepStores` is called from `Topology.Remove` after a successful teardown; the probe is a parameter of the housekeeping call so `Remove`'s signature can carry it from both callers.
