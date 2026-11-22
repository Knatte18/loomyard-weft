---
round: 1
focus:
  - "Sandbox exception. Every mutating `lym fabric` verb (add, push, pull, sync, checkout, remove, cleanup --apply, prune --apply, reconcile, clone, hand-over, replay) must run inside the round's own `.lym/harden-sandbox` hub and stay refused against the real hub and against another worktree's sandbox root. Failure: a verb mutates outside the sandbox root, or the refusal has no named way forward. Repro: make a one-repository hub with `lym harden sandbox new --single-repo`; run each verb with the printed `.dev-bin` lym from inside it (expect success); run the same verb with cwd at this worktree's real hub and at a sibling worktree's sandbox hub path (expect refusal, no ref or file change); try a symlinked or `..`-relative path into the root and a hub moved out of the root. Settle: is the guard keyed on the resolved hub path, and can any verb or helper reach another hub's store?"
  - "Joining. A second `lym fabric clone <url>` (no `--single-repo`) of a seeded one-repository origin must take the one-repository layout: records store fetch refspecs `+refs/lym/heads/*:refs/remotes/origin/*` and `+refs/lym/tags/*:refs/tags/*`, push refspecs into `refs/lym/`, `remote.origin.tagopt --no-tags`, `lym.primeRecordsBranch`, and `branch.main.merge refs/lym/heads/main`; it must see the board and archive tags. Failure: identity layout read, a missing upstream, a board that is empty or stale, or a code store holding a ref from the records namespace. Repro: seed hub A with `--single-repo`, land a task so origin holds an archive tag, clone B into a fresh directory under the sandbox root with a different `--into`; compare `git config --get-regexp` of both records stores, run `lym board list`, `lym fabric status`, `lym fabric pairs` in B. Also join an origin that is mid-seed, one whose board branch is the legacy `lym-bolt`, and one whose default branch holds a warp binding."
  - "Namespace leaks. No verb may create a records ref under plain `refs/heads/` or `refs/tags/` on origin, and none may read a code ref as a records ref. Failure: an origin-side ref outside `refs/lym/` that holds records, or a records branch pushed by a default push with an identity namespace. Repro: after each of add, push, pull, sync, replay, hand-over, checkout, remove, teardown, reconcile, `cleanup --apply` and `prune --apply`, list origin with `git ls-remote <origin>` and classify every ref; run each verb from a task worktree, from the prime and with a stale or missing fetch refspec (the layout read must refuse with a way forward, never fall back to identity). Settle which gitrepo and fabricengine call sites build a ref from a literal `refs/heads/` or `refs/tags/` on a records-store path, with an enumeration that states what the search cannot see."
  - "Archive tags. The archive tag of a discarded task branch, a landed pair and a removed run must land under `refs/lym/tags/archive/<slug>/<sha12>`, never plain `refs/tags/`, and never a tag named for the code store's origin. A tag of the code store (`archiveTaskTip`) and a tag of the records store (`tagWeftTipForArchive`) share the shape; the code store has the identity namespace unless its origin is read as the records layout. Failure: a plain `refs/tags/archive/...` on origin, a local tag left that a later run reuses against another tip, or a second push that reports success on a refused tag. Repro: discard a task branch (`lym fabric remove` of a pair with unpushed and pushed commits, then `lym fabric cleanup --apply`), land a task, abandon a task; after each, `git ls-remote --tags <origin>` and `git ls-remote <origin> 'refs/lym/tags/*'`. A sandbox trial once saw one in plain `refs/tags/`: find which verb and which store."
  - "Two clones at once. Two clones of one origin, each pushing and pulling, each writing the board, and one lease that loses. Failure: a lost lease reported as success or as a bad token, a board write lost without a record, a push that overwrites the other clone's records branch, or a refusal with no way forward. Repro: clone A and B of one origin; write a board entry in each, then `lym board sync` in A and in B (the second must fetch, rebase or refuse honestly); add a task of the same slug in both; push the prime's records branch from both; race `lym fabric push` from both in the foreground one after the other with the first one's push held (`kill -STOP`) and then released. Run the same drive sequentially as a control, then from a second pair of hubs beside the first."
  - "Remote delete. lym's own delete of a namespaced branch (`gitrepo.DeleteRemoteBranch`, `DeleteRemoteBranchLeased`, and the teardown and reconcile paths that call them) of a missing branch must record no deletion, and of a present one must remove `refs/lym/heads/<name>` and nothing under `refs/heads/`. Failure: a deletion recorded for a ref that was never there, a code branch of the same name deleted, a leased delete that deletes a ref another clone moved. Repro: with a records branch and a code branch of the same short name on origin, run the teardown and `cleanup --apply` on a pair; check both refs; delete a branch by hand first and rerun; move the branch from clone B between A's read and A's delete."
  - "A pair verb killed mid-operation. After `kill -9` of `lym fabric add`, `push`, `sync`, `checkout`, `remove`, `pull` and `reconcile` at each phase (before the records push, between the records push and the code push, after both, during the archive tag push, during the teardown lock hold), the hub and origin must hold a state that the verbs a reader runs (`status`, `pairs`, `list`, `reconcile`, `prune`, the same verb again) name a way forward from. Failure: an orphan ref on origin or a worktree with no verb that clears it, a stale teardown lock or remote-refs lock that blocks forever, a record that reports a push as done that was not, or a way forward that loops. Repro: use the `seedPushHook`-style pauses where a verb has none by sending `kill -STOP` to the verb at a point found with `strace`-free means (a large pack, a pre-push hook that sleeps in the code store), then `kill -9`; read origin and the hub; run each reader verb; rerun the killed verb."
  - "A killed seed and its way forward. After a `lym fabric clone --single-repo` is killed with `kill -9` at each point of `pushCloneSeed` (before the records push, after it and before the board, after the board and before the upstream config), `lym board sync` and `lym fabric clone --single-repo <url> --reset` must lead to a working hub, and no refusal may loop back to a refusal it already gave. Failure: sync refusing with a sentence that is false for the state (the board seeded, or not), a one-URL clone refused because origin holds a board, a `--reset` that removes a hub whose origin ref it then cannot clean, or advice whose commands do not work when typed. Repro: a pre-receive or pre-push hook in the bare origin that sleeps lets the kill land after origin's receive; kill, then read `git ls-remote <origin> 'refs/lym/*'`, run `lym board sync`, run each printed command exactly, then the `--reset` clone and the join clone. Fixed by `seed-killed-way-forward-loops` (commit 0e4782373): re-drive it first, then vary the origin's state (board of another clone, board with no common history, records branch at another tip)."
  - "Namespace cache and layout read. `originNamespaceAt` caches an identity answer for the process and answers identity for a directory with no `.git` entry. A long-lived process (a reed server, a dobby, a `lym fabric` verb that opens a store before a clone wires it) must not keep a stale answer. Failure: a records store read as identity after its refspecs were wired, a layout read that errors on a hub in a transient state, or a records push into `refs/heads/`. Repro: start the hub's reed server, rewire a store's fetch refspecs and delete the prime key by hand, then run a push; run a verb during a clone of a second hub; remove `remote.origin.fetch` from the records store and run every pushing verb."
  - "Reconcile, prune and cleanup against origin. `lym fabric reconcile`, `cleanup --apply` and `prune --apply` judge local branches against origin and act on `refs/lym/heads/` and `refs/heads/` as separate namespaces. Failure: a records branch judged against a code branch of the same name, a remote-only run torn down although the code branch is live, an ended entry torn down although another machine owns it, or a delete of an archive tag. Repro: build tasks in each state (live, landed, abandoned, owned by another clone, origin-only), run each verb judge-only and then moving; read origin and the hub after each. Sabotage the namespace in one call site to prove the drive enters it."
---

## Modules and focus

The campaign hardens `./internal/fabricengine`, `./internal/fabriccli` and `./internal/gitrepo`, on the one-repository hub layout alone.
The operator's focus, from the decision record's `## Goal`:
the second hardener campaign on the layout that commit 57015a52 added.
One origin holds the code under `refs/heads/` and `refs/tags/`, and the records and board under `refs/lym/heads/` and `refs/lym/tags/`.
The focus binds: no round reviews outside it, and no focus file steers a round outside it.

The first campaign, `harden-fabricengine` (landed 37649c772), could not drive the mutating `lym fabric` verbs, since a task session was refused them.
Commit 4037bdf36 lets a hardener round run every fabric verb inside its own `.lym/harden-sandbox` hubs.
This campaign drives what the first left unverified, with real git and real `lym` verbs, on hubs from `lym harden sandbox new --single-repo`.
It re-drives the first campaign's fixes (F1 to F8, and 37649c772 and 0e4782373 after it) only to check they still hold.

In scope:

- the sandbox exception of the mutating verbs;
- joining a seeded origin with a second one-URL clone;
- namespace leaks on add, push, pull, sync, replay, hand-over, checkout, remove, teardown, reconcile, `cleanup --apply` and `prune --apply`;
- archive tags of a discarded task branch, of a landed pair and of an abandoned run;
- two clones of one origin, with a board write from each and a lease that loses;
- lym's own remote delete of a namespaced branch;
- a pair verb killed mid-operation;
- a killed seed and its way forward (`seed-killed-way-forward-loops`).

Code paths in scope, in `internal/fabricengine`:
the layout and namespace (`recordslayout.go`, `recordsgit.go`, `recordsprobe.go`, `origin*.go`),
the clone, seed and teardown of a seeded clone (`clone.go`, `destroy.go` `pushCloneSeed`, `seedlocalbares.go`),
the push and pull paths (`pushclassified.go`, `pushanchored.go`, `failedpush.go`, `pull.go`, `hubpull.go`, `handover.go`, `replay.go`, `boltreplay.go`),
the teardown, archive and delete paths (`remoteteardown.go`, `archive.go`, `ownerarchive.go`, `taskbranchdiscard.go`, `teardownlock.go`, `cleanupremotewarp.go`, `prune.go`, `cleanup.go`, `remove.go`),
and reconcile (`reconcile.go`, `originreconcile*.go`).
In `internal/gitrepo`: `refnamespace.go`, `push.go`, `pushdestination.go`, `pushretry.go`, `remote.go`, `remoterefslock.go`, `fetch`-side reads in `refreads.go`.
In `internal/fabriccli`: the verb layer in `fabric.go`, `clone.go`, `hubpull.go`, `remove.go`, `roleguard.go` and the sandbox refusals in `cli_refusals_test.go`.

## What to read

Read first, in this order:

1. `internal/fabricengine/recordslayout.go` and `internal/gitrepo/refnamespace.go`, the only places the layout and namespace are spelled.
2. `internal/fabricengine/clone.go` and `pushCloneSeed` with its helpers in `internal/fabricengine/destroy.go`, then `refuseUnseededBoard` in `pushclassified.go`.
3. `internal/fabricengine/archive.go`, `remoteteardown.go`, `taskbranchdiscard.go`, and `DeleteRemoteBranch` and `DeleteRemoteBranchLeased` in `internal/gitrepo/push.go`.
4. `internal/fabricengine/doc.go` and `internal/gitrepo/doc.go`, then `internal/fabriccli/doc.go`.
5. `docs/overview.md` and `docs/getting-started.md` for the one-repository layout, `PATTERN.md` entries that touch fabric, and `docs/research/single-repo/*/report.md` (three trials of the layout, which settled the happy path and filed note B2).
6. The verbs' own `--help`: `lym fabric --help` and each verb below it, `lym harden sandbox --help`, `lym harden sandbox new --help`.
7. Existing tests as a map of what is already pinned and so needs no rerun as a review:
   `clone_singlerepo_integration_test.go`, `clone_singlerepo_killedpush_integration_test.go`, `onerepository_lifecycle_integration_test.go`,
   `recordslayout_integration_test.go`, `recordslayout_refnames_integration_test.go`, `remoteteardown_namespace_integration_test.go`,
   `archive_integration_test.go`, `cleanupremote_integration_test.go`, `internal/gitrepo/deleteremotebranch_integration_test.go`
   and the `livestate_*` matrix.

## Layouts

A round drives the one-repository hub that `lym harden sandbox new --single-repo` builds: one local bare origin holding the code on its branches and the records and board under `refs/lym/`.
That is the layout the focus names, and the only layout every scenario runs on.

The two-repository hub that `lym harden sandbox new` builds is out of scope as a subject.
A round makes one only as a control, where a scenario needs the identity namespace to prove that a leak check sees a leak (a namespace sabotage on the one-repository hub is the better proof where it works).

Two clones of one origin (joining, two clones at once) are both one-repository hubs:
make the second by `lym fabric clone <url> --into <dir>` from inside the sandbox root, against the first hub's origin path.
Every hub, the second clone included, lives under the sandbox root and is cleared by `lym harden sandbox clear`.

## Out of scope

- The two-repository layout and everything in fabric that is not on a one-repository path: any behaviour of a two-repository hub is not a finding of this campaign.
- Every module outside `internal/fabricengine`, `internal/fabriccli` and `internal/gitrepo`, board, batten, webster, reed and loom internals included, except where a fabric verb's result is read through them (`lym board list`, `lym board sync`).
- The first campaign's closed findings F1 to F8: check that they hold, never re-open them as new findings unless a drive shows one undone.
- A real GitHub remote: no sandbox hub has one.
  Branch rules, a 403 or a rate limit from a host cannot be driven here; name them under Coverage, never as a finding.
- Correct absences, never flagged:
  a branch with no origin tracking ref is not judged by `ReconcileOrigin`;
  a judge-only reconcile writes no `FETCH_HEAD`;
  `main-weft` does not advance on landing;
  a one-URL `--single-repo` clone is refused when origin already holds a board, with `lym fabric clone <url>` as its way forward;
  the code store of a one-repository hub holds no ref from the records namespace;
  a `lym fabric` mutating verb run outside the sandbox root, from a task session, is refused by design.
- A timeout under the concurrent copies' CPU peg is a diagnostic, never a defect.
- `lym selfreport create` is never run.

## Live-substrate cost declaration

No live verb or test of these modules spawns a real LLM session.
`lym fabric` verbs, `lym harden sandbox` and the modules' own integration tests drive git, `lym` processes and the filesystem on throwaway local bare repositories.
Nothing in them requires tier 3, so there is no execution ban.
A round that runs a `lym beam` or `lym batten` run on a sandbox hub to produce a task state starts a real LLM session; the charter does not ask for one:
build task states with `lym fabric add`, `lym board` verbs and plain commits.
A round that finds it needs a real session names why first, and runs one at a time.

## Teardown check

Run at the end of the round, and record every output:

```
lym harden sandbox clear
lym harden sandbox list
pgrep -af 'harden-sandbox' || true
pgrep -af 'lym (reed|fabric|board)' || true
ls .lym/harden-sandbox 2>/dev/null || true
git -C . worktree list
git -C . status --short
git -C . stash list
```

The first command ends what the hubs left running and empties the root.
`lym harden sandbox list` must print no hub.
The `pgrep` lines must show no process that names the sandbox root (a killed or stopped verb, a reed server, a sleeping hook in a bare origin).
No directory may stay under `.lym/harden-sandbox`, and no worktree, stash or modified file other than the round's review file and the fix row's commits may remain.
Remove each scratch file by its literal path under `.scratch/`.

## Round context

Round 1 assignment: a full adversarial review of the scope above, on the one-repository layout.
Walk every focus entry and record OK, WARN or FAIL for each, then improvise further scenarios of your own,
in particular:

- verbs combined in orders nothing has tried (push in a hub whose second clone just landed a task, `remove` right after `sync`, `reconcile` between a push and its archive tag);
- orphans of an operation that failed midway (a records branch on origin with no code branch, a code branch with no records branch, an archive tag with no landed run, a leftover local tag that a retry reuses);
- dead-but-present state (a lock file whose holder is gone, a worktree whose records junction is dangling, a reed server of a removed hub);
- rapid churn (add and remove of the same slug in a loop, from one clone and from two);
- scope across hubs and worktrees (two sandbox hubs of this worktree sharing a shortname, a hub of another worktree's sandbox root).

Re-drive the first campaign's fixes (37649c772, 0e4782373) once as a still-holds check, with the sabotage proof of the rubric.
Prove each scenario reached the mechanism it claims to exercise: sabotage the namespace, the lease or the hook, and watch the drive fail.
Give every finding its file and line, a concrete failure scenario, a severity, a suggested fix and the repro steps that showed it.
Every finding is recorded and fixed, large or small.
Deploy first: build through `lym harden sandbox new --single-repo`, and invoke only the `.dev-bin` lym it prints.
