---
verdict: BLOCKING
findings:
  - id: G1
    carries: F2
    severity: BLOCKING
    class: design
    location: internal/fabricengine/archive.go:254
    summary: "The archive tag of a discarded task branch still lands in plain refs/tags/ on a one-repository origin when the code store spells origin's URL differently from the records store. archiveTagNamespaceAt takes the records namespace only when the two remote.origin.url strings are equal, and falls back to the identity namespace otherwise"
    fix: "In archiveTagNamespaceAt (internal/fabricengine/archive.go:252-256), stop falling back to the identity namespace when the two stores' origin URLs differ as strings. Return the records store's namespace whenever the prime's records sibling reads the one-repository namespace through originNamespaceAt, and drop the codeURL and recordsURL comparison. When a URL check is kept, a mismatch must return an error that names both URLs and the way forward (set the code store's URL to the records store's with git remote set-url), never the identity namespace. Add a row to the one-repository discard test in internal/fabricengine/taskbranchdiscard_integration_test.go: with the code store's origin URL respelled as the file:// form of the same bare path, DiscardTaskBranch leaves refs/lym/tags/archive/<slug>/<sha12> on origin and no ref under refs/tags/."
  - id: G2
    carries: F4
    severity: MEDIUM
    class: design
    location: internal/fabricengine/pushclassified.go:140
    summary: "With the records store's fetch refspecs removed, lym fabric reconcile still fails with the raw text 'gitrepo - resolve upstream refs/remotes/origin/lym/heads/main - reference not found' and names no way forward. The site is Bolt.pullFirst at internal/fabricengine/bolt.go:319, which reads the board's upstream with no layout read first; the push and pull sites round 1 fixed hold"
    fix: "In Bolt.pullFirst (internal/fabricengine/bolt.go, before the repo.Fetch call at line 313), call originNamespaceAt(b.path) and return its error, as pushClassified (pushclassified.go:140) and pullWeftArm (pull.go:329) do since round 1, so the board pull refuses with the records-store message and its restore commands. Add a test beside internal/fabricengine/boltpullfirst_integration_test.go over hubforge's one-repository builder: unset remote.origin.fetch in the records store, run PullThenCommitWritten, the call lym fabric reconcile makes, and assert the error names the config --replace-all remote.origin.fetch restore command and holds no 'resolve upstream' text."
  - id: G3
    severity: BLOCKING
    class: design
    location: internal/fabricengine/pendingteardown.go:97
    summary: "The re-run a pending-teardown record names finishes the teardown without the archive the killed child owed. After a kill before the records branch push, running lym fabric remove <slug> --remote again archives origin's tip alone and deletes the records branch, leaving the local-only records commit under a local tag origin never gets. After a killed discard, the re-run deletes the task branch on origin with no archive tag of its unlanded commit on origin. The broken sites are archiveWeftTip with an empty job tip (archive.go) and checkUnlandedRemoteTip counting local tags (destroy.go:1046)"
    fix: "Make the re-run push what the first run tagged before any origin delete. In runRemoteTeardown (internal/fabricengine/remoteteardown.go), before the archive and delete steps of every job kind, list the local tags named archive/<slug>/* in the records store (remove and cleanup jobs) and in the code store (discard jobs, and remove jobs with DeleteTaskOnOrigin), list origin's with remoteTagsUnder, and push each tag origin lacks with pushArchiveTag. A failed push stops the job with an error wrapping ErrArchiveFailed, so no origin delete follows. Also carry the job's WeftTip, TaskTip and Worktree in the PendingTeardown record, which writePendingTeardown already receives, and have pendingTeardownWayForward (pendingteardown.go:97) name the tag push first, in the wording failedRemoteSteps uses, for a record whose tip is set. Add two tests over hubforge's one-repository builder. One - a removal whose local records tip origin lacks, with the remote job dropped after the local half and Remove then run again with remote set, leaves refs/lym/tags/archive/<slug>/<local sha12> on origin. Two - a discard whose remote job is dropped, followed by the removal and the discard again, leaves the archive tag of the task tip on origin whenever the task branch is deleted there."
  - id: G4
    severity: BLOCKING
    class: design
    location: internal/fabricengine/remoteleftover.go:279
    summary: "Since archiveTagNamespaceAt, both stores of a one-repository hub push and read their archive tags under one origin prefix with no side in the name, so each store's reader takes the other store's tags. lym adopt --archives root reports every archive-owner tag twice, a records commit as a code-side archive of branch main and a code commit as a records-side archive of main-weft, and it fetches the records commit into the code store under a local tag. A second clone's records store also fetches every code-side archive tag with its code commits"
    fix: "Give the code store's archive tags their own prefix under the records namespace root, so they stay out of plain refs/tags/ and out of the records store's refs/lym/tags/. Declare one more constant beside RecordsNamespaceTags in internal/fabricengine/recordslayout.go, for example RecordsNamespaceCodeTags with the value code-tags/, and have archiveTagNamespaceAt (internal/fabricengine/archive.go) return, for a code store of a one-repository hub, a namespace whose Tags prefix is RecordsNamespaceRoot plus that constant, and for a records store the store's own namespace as now. pushArchiveTag, remoteTagsUnder (remoteleftover.go:279), readSideOwnerArchives (ownerarchive.go:317) and the way forward in failedRemoteSteps then address each side's own prefix with no further change. Extend internal/fabricengine/remoteteardown_namespace_integration_test.go: after an owner archive of both sides of a one-repository pair, ReadOwnerArchives returns each tag once with its own side, the code store holds no tag at a records commit, and a fetch in the records store brings no code commit."
prior:
  - id: F1
    status: resolved
    answer: "Re-driven in P3 over own, symlinked, moved, relative and sibling paths; every path outside the role's own sandbox root is refused"
  - id: F2
    status: open
    answer: "Holds while both stores spell origin's URL the same; reproduced again with the code store's URL respelled, carried as G1"
  - id: F3
    status: resolved
    answer: "The refusal lists the one local commit git counts, P5"
  - id: F4
    status: open
    answer: "The push site holds; lym fabric reconcile still returns the raw gitrepo error through Bolt.pullFirst, carried as G2"
  - id: F5
    status: resolved
    answer: "The pull refuses with the restore commands and moves nothing, P4"
  - id: F6
    status: resolved
    answer: "status and prune list the killed teardown in every phase driven, P6; what the named re-run then does is raised as G3"
  - id: F7
    status: resolved
    answer: "The refusal names lym harden sandbox new, P3"
---

### [BLOCKING] G1 — a respelled origin URL sends the code-side archive tag back to plain refs/tags/

**Location:** `internal/fabricengine/archive.go:254`

**Issue:** CONFIRMED, probe P2. `archiveTagNamespaceAt` compares the two stores' `remote.origin.url` strings and returns the identity namespace when they differ.
A one-repository hub whose code store names origin as `file://<path>` and whose records store names it as `<path>` is still one origin.
An operator who switches the code store between https and ssh forms reaches the same state.
After `git -C $H1/origin-bare remote set-url origin "file://$O1"`, `lym batten cancel t2 --remove` left `refs/tags/archive/t2/36b86e577f27` on origin beside `refs/lym/tags/archive/t2/863c3c45d44d`.
`lym fabric status` listed nothing. The same drive with equal URLs (P1) put both tags under `refs/lym/tags/`.
This is F2's failure, the decision record's "never plain `refs/tags/`", reached through the condition the fix added.

### [MEDIUM] G2 — reconcile still answers a raw gitrepo error on an unreadable records layout

**Location:** `internal/fabricengine/pushclassified.go:140` (the fix site is `Bolt.pullFirst`, `internal/fabricengine/bolt.go:319`)

**Issue:** CONFIRMED, probe P4. With `remote.origin.fetch` unset in the records store, `lym fabric reconcile` from a task worktree and from the prime answers
`{"error":"gitrepo: resolve upstream refs/remotes/origin/lym/heads/main: reference not found","mutations":[],"ok":false}`.
`Bolt.pullFirst` runs `repo.Fetch()`, which succeeds with no refspec, and then `repo.UpstreamSHA()`, whose error it returns as it is.
Push, sync, pull, checkout, add, remove, cleanup and `board sync` all answer the records-store message with its restore commands; reconcile is the verb the round-1 fix did not reach.
After the restore commands were typed, the same `reconcile` answered its normal envelope.

### [BLOCKING] G3 — the re-run a pending teardown names deletes on origin what the killed child had not archived there

**Location:** `internal/fabricengine/pendingteardown.go:97`

**Issue:** CONFIRMED twice, probe P6.
Records side, slug `k1`: the pair held one unpushed records commit, `a603ad1`, when `lym fabric remove k1 --remote` ran and its child was killed in the records branch push.
`status` and `prune` named `run lym fabric remove k1 --remote again`. That re-run has no local branch left, so its job carries no tip:
it archived origin's tip `5a85cd3a` and deleted `refs/lym/heads/k1-weft`. Origin holds no ref at `a603ad1`; the commit sits under the local tag `archive/k1/a603ad17c686` alone, and no reader names it any more.
The control with no kill (`k6`) put the local tip's tag on origin.
Code side, slug `k7`: the discard's child was killed in the push of `archive/k7/d2e6791ce231`. The record named `run the command that discarded the task branch again`.
`lym batten cancel k7 --remove` again deleted `refs/heads/k7` on origin and pushed no tag: the removal's landed check passed because the local tag the first run made holds the commit,
and the discard that followed found no branch to archive. Origin lists `refs/lym/tags/archive/k7/53b7f22df05d`, the records tag, and nothing at `d2e6791c`.
`PATTERN-fabric-boundary` says the child "pushes the tag to the weft origin before any delete on origin, so the run records stay reachable"; the re-run the record names breaks that order.
The failed-step notice of the same teardown (`failedRemoteSteps`) names the tag push before the re-run; the pending record's way forward drops it, and the record holds no tip to name it with.

### [BLOCKING] G4 — code-side and records-side archive tags share one origin prefix, and each side reads the other's

**Location:** `internal/fabricengine/remoteleftover.go:279` (with `internal/fabricengine/ownerarchive.go:317` and `archiveTagNamespaceAt` in `archive.go`)

**Issue:** CONFIRMED for the read, probe P12; the two tags were pushed by hand to the destination `pushArchiveTag` computes, and the read was the real verb.
With one `archive-owner/root/<sha12>` tag pushed from each store to `refs/lym/tags/`, `lym adopt --archives root` listed four entries:
`9c498c15db68` (a code commit) as side `code` and as side `records` on `main-weft`, and `dc6f39e372cd` (a records commit) as side `records` and as side `code` on `main`.
After the read the code store held the local tag `archive-owner/root/dc6f39e372cd` at the records commit, and the records store held the code commit.
The charter lists "the code store of a one-repository hub holds no ref from the records namespace" as an invariant; this read breaks it.
A commit of one store is never contained in the other store's branch, so each false entry stays listed for as long as the tag exists.
The same sharing shows in P1: the second clone's records store fetched `archive/t1/82c68e8ca904` with the code commit it names,
and `remoteArchiveTags`, `SlugRecordsTags` and `hasWeftEvidence` now read a discarded task branch's code tag as a records tag of the slug.
Before round 1 each store pushed and read under its own namespace, so no reader crossed sides.

## Method

Reviewer model: `claude-fable-5-2`, which differs from round 1's `opus`.
Binary: `/home/knatte/Code/loomyard-LYMHUB/harden-fabric-single-repo-verbs/.dev-bin/lym`, built by `lym harden sandbox new --single-repo` from HEAD `8a8b8debb`.
Every hub below lives under `.lym/harden-sandbox/`; `$L` is the `.dev-bin` lym, `$H1` the first one-repository hub, `$O1` its bare origin.
No source file was edited, and no prior round's review or fixer report was opened before the findings below were written.

## Probe log

### P1 — archive tag of a discarded task branch (focus entry 1)

Commands, in `$H1` (`hub-2983790946/origin-bare-LYMHUB`):

```
(cd $H1/origin-bare && $L fabric add t1)
(cd $H1/t1 && echo hello > real.txt && git add real.txt && git commit -qm "t1: real file" && $L fabric push)
git ls-remote $O1
(cd $H1/origin-bare && $L batten cancel t1 --remove)
git ls-remote $O1
(cd $H1/origin-bare && $L fabric cleanup --apply)
git ls-remote $O1
```

Observed after add and push: `refs/heads/main`, `refs/heads/t1` at `82c68e8c`, `refs/lym/heads/main`, `refs/lym/heads/main-weft`, `refs/lym/heads/t1-weft` at `4c88c804`.
Observed after `batten cancel t1 --remove`: `refs/heads/t1` and `refs/lym/heads/t1-weft` gone;
`refs/lym/tags/archive/t1/4c88c804917a` (records tip) and `refs/lym/tags/archive/t1/82c68e8ca904` (code tip) present; no ref under `refs/tags/`.
`cleanup --apply` changed nothing on origin.
Local tags: the code store holds `archive/t1/82c68e8ca904`, the records store `archive/t1/4c88c804917a`.

Second clone: `(cd $S/hub-2983790946 && $L fabric clone $O1 --into $S/hub-2983790946/b)` made `$H1B`.
Its code store lists no tag; its records store lists `archive/t1/4c88c804917a` and `archive/t1/82c68e8ca904`.
`git -C $H1B/origin-bare-weft cat-file -t 82c68e8c…` answers `commit`: the records store of the second clone now holds the code commit and its tag.
`git config --get-regexp` of B's records store equals A's: both fetch refspecs, both push refspecs, `tagopt --no-tags`,
`branch.main.merge refs/lym/heads/main`, `branch.main-weft.merge refs/lym/heads/main-weft`, `lym.primerecordsbranch main-weft`.

Control, two-repository hub `$H2` (`hub-1078206708/warp-bare-LYMHUB`, from `lym harden sandbox new`), same drive with slug `t1`:
the code origin holds `refs/tags/archive/t1/e0bea81d5239` and the records origin `refs/tags/archive/t1/4d5b84935f9c`.
The control differs from the one-repository result, so the one-repository drive entered `archiveTagNamespaceAt`.

Verdict on the charted claim: OK for a hub whose two stores spell origin's URL the same.

### P2 — the same discard after the code store's origin URL is respelled (improvised)

`archiveTagNamespaceAt` (`internal/fabricengine/archive.go:252-256`) takes the records namespace only when the two stores' `remote.origin.url` strings are equal.

```
(cd $H1/origin-bare && $L fabric add t2)
(cd $H1/t2 && echo hello > real2.txt && git add real2.txt && git commit -qm "t2: real file" && $L fabric push)
git -C $H1/origin-bare remote set-url origin "file://$O1"
(cd $H1/origin-bare && $L batten cancel t2 --remove)
git ls-remote $O1
```

Observed on origin: `refs/lym/tags/archive/t2/863c3c45d44d` (records tip) and `refs/tags/archive/t2/36b86e577f27` (code tip).
The code-side archive tag is under plain `refs/tags/` on a one-repository origin. `lym fabric status` lists nothing. FAIL, finding N1.
The URL was put back with `git -C $H1/origin-bare remote set-url origin $O1` after the probe.

### P3 — the sandbox guard for pushes and the other mutating verbs (focus entry 6)

The session's strand name, as the refusals print it, is `ly:harden-fabric-single-repo-verbs:harden-burler-review`.
All commands ran with cwd `$W`, the real hub's task worktree, unless a line says otherwise. `$M` is hub B moved to `$W/.scratch/moved/`.
Origin's refs were digested with `git ls-remote $O1 | sha1sum` before and after the refused calls: `6637df30563d` both times.

| Case | Command | Observed |
| --- | --- | --- |
| (a) own sandbox pair | `$L fabric push --weft-path $H1/t3-weft --warp-path $H1/t3` | `ok: true`, `branch_pushed` t3; origin `refs/heads/t3` moved to `0852e71d` |
| (c) symlink in `.scratch` into the root | `ln -s $H1/t3 $W/.scratch/link-t3` (and `-weft`), a new commit, push with the link paths | `ok: true`; origin `refs/heads/t3` at `7a26a114`, the pair's HEAD |
| (b)/(f) hub moved out of the root | `mv $BDIR/origin-bare-LYMHUB $W/.scratch/moved/`; push with `$M/origin-bare-weft` and `$M/origin-bare` | refused, text names `lym harden sandbox new` |
| (f2) one flag only, outside | `$L fabric push --weft-path $M/origin-bare-weft` | refused |
| (f3) records path inside, code path outside | `--weft-path $H1/t3-weft --warp-path $M/origin-bare` | refused |
| (d) `..`-relative path leaving the root | `--weft-path .lym/harden-sandbox/../scratch/moved/…-weft --warp-path .lym/harden-sandbox/hub-2983790946/../../scratch/moved/…` | refused |
| (c2) symlink inside the root pointing out | `ln -s $M/origin-bare $S/out-link` (and `-weft`); push with the links | refused |
| (e) a sibling worktree's root | paths under `/home/knatte/Code/loomyard-LYMHUB/later-transition-push-wait/.lym/harden-sandbox/hub-1/…` (no such root exists on this machine, so the paths do not exist) | refused |
| path flag on another verb | `$L fabric pull --weft-path $H1/t3-weft --warp-path $H1/t3` | `lym fabric pull refuses the path flags that only a detached push passes` |

From `$W`, each of `add`, `remove`, `checkout`, `clone --into $W/.scratch/clone-zz`, `pull`, `sync`, `reconcile`, `cleanup --apply`, `prune --apply`, `unwire`, `commit`,
`shortname zz`, `merge`, `anchor --set` answered the task-session refusal; `status`, `prune`, `cleanup`, `list`, `pairs` and `shortname` with no argument ran.
`git -C $W status --short` stayed empty and HEAD stayed `8a8b8debb`.
Clone destination: from a sandbox hub, `--into $W/.scratch/clone-zz` and `--into ../scratch/clone-zz` are refused; from `$W`, `--into $S/clone-zz` passes the guard and `--into .` is refused.
Control that the guard is reached and passes: case (a), and every verb of P1 and P2 run from inside `$H1`.
The links were removed by literal path and hub B was moved back.

Verdict: OK. F1 and F7 hold.
A sibling worktree with a real sandbox root was not available, so case (e) rests on the path rule alone.

### P4 — a records store whose fetch refspecs are removed (focus entry 3)

Fixture: `$H1` with the pushed pair `t3`; `git -C $H1/origin-bare-weft config --unset-all remote.origin.fetch`.
Origin's `refs/lym/heads/t3-weft` was advanced by hand from a plain `git clone` under the sandbox root before the pull.

| Verb, cwd | Observed |
| --- | --- |
| `fabric push`, `t3` | `ok: false`, the records-store message with both restore commands, the code side withheld; `fabric status` lists `t3-weft` class `other` with the same text |
| `fabric sync`, `t3` | `ok: true`, two `push_spawned`; six seconds later `fabric status` lists the `t3-weft` failed push, count 4 |
| `fabric pull`, `t3` | `ok: false`, the records-store message, `mutations: []` |
| `fabric checkout t3`, `t3` (same branch) | `ok: true`, no remote step |
| `fabric checkout onlyorigin`, `t3` (branch only origin holds) | `ok: false`, the records-store message; both worktrees stayed on `t3` and `t3-weft` |
| `fabric reconcile`, `t3` and the prime | `{"error":"gitrepo: resolve upstream refs/remotes/origin/lym/heads/main: reference not found","mutations":[],"ok":false}` — FAIL |
| `board sync`, prime | `ok: false`, `sync push:` then the records-store message naming `$H1/_bolt` |
| `fabric push` and `fabric pull`, prime | `ok: false`, the records-store message; status lists `main-weft` class `other` |
| `fabric add t4`, prime | `ok: false`, the records-store message, no pair made |
| `fabric cleanup --apply --remote`, prime | `ok: false`, `cannot read the records store` with the message |
| `fabric remove t4 --remote`, prime | `ok: false`, the records-store message |
| `fabric prune --apply`, prime | `ok: true`, nothing to do |

No verb wrote a ref under `refs/heads/` on origin: the only refs that changed were the two `onlyorigin` refs made by hand for the checkout probe.
The hub log holds one `push failed and was recorded` Warn per records push attempt, and two for the one detached push of the sync, which is why the count rose by two per push.

Restore, typed exactly as the refusal printed them (the `t3-weft` spelling):
`git -C <…>/t3-weft config --replace-all remote.origin.fetch '+refs/lym/heads/*:refs/remotes/origin/*'` and
`git -C <…>/t3-weft config --add remote.origin.fetch '+refs/lym/tags/*:refs/tags/*'`; both exit 0, and the store then lists both refspecs.

Control after the restore: `fabric push` in `t3` pushed both sides (the records side replayed over the hand-made origin commit: `0c38b42 weft sync` over `24795aa other machine`);
`fabric reconcile` answered its `hub_config` envelope; `board sync` answered `ok: true`; the prime's `fabric push` landed and `failed_pushes` emptied.
The control `fabric pull` in `t3` advanced the records side and then answered `warp remote diverged and local warp has unpushed commits; aborting, no changes`
for a code branch that was one commit ahead of an unmoved origin. That text is `pullWarpArm`'s for both layouts, so it is outside this campaign's focus; it is noted, not raised.

Verdict: WARN. F5 holds. F4's push site holds, and the same raw `gitrepo:` error with no way forward still reaches `lym fabric reconcile`
through `Bolt.pullFirst` (`internal/fabricengine/bolt.go:319`), which reads the board's upstream with no layout read first. Carried as F4.

### P5 — a rejected first push (focus entry 4)

Origin `$O1` got a `pre-receive` hook that declines every push while a flag file exists.
With the flag set, `fabric add s1` ran in clone A (`$H1`) and in clone B (`$H1B`); origin held no `s1` ref and A's status listed `s1-weft` class `other`.
With the flag gone, A committed and ran `fabric push`: origin `refs/heads/s1` and `refs/lym/heads/s1-weft` landed.
B committed and ran `fabric push`:

- `ok: false`; `origin/s1-weft holds commits this branch lacks (local commits: e0b133ae98af89710fcda8ab69e75a8be398cd22)`, then the `git reset --keep` and `git merge` advice.
- `git -C $H1B/s1-weft rev-list --count origin/s1-weft..HEAD` answers `1`, and the one commit is `e0b133a fabric: record parent branch for s1`.
- `git config --get-regexp 'branch\.s1'` in B's records store lists nothing: the branch has no upstream.
- `fabric status` in B lists `s1-weft` class `not_fast_forward` with the same commit.

The advised `git merge origin/s1-weft` typed in `$H1B/s1-weft` merged cleanly, and `fabric push` then landed the records side under `refs/lym/heads/s1-weft`
and refused the code side with `origin/s1 holds commits this branch lacks; run git merge origin/s1 …`, recorded as class `not_fast_forward`.
No write of either clone was lost and no ref left the namespace.

Verdict: OK. F3 holds.

### P6 — a teardown child that dies (focus entry 5)

Origin's hook was replaced by an instrumented one: it appends `<time> upd|del <ref>` to `$O1/hook.log` for every ref update,
sleeps while the update matches the pattern in `$O1/sleep-pattern`, and declines after the sleep when `$O1/decline` exists.
The decline keeps the killed child's orphaned `git push` from landing after the kill, so origin holds what a crash at that step leaves.
Each drive: `fabric remove <slug> --remote` from the prime, wait for `$O1/sleeping`, `kill -9` of `lym fabric remote-teardown --slug <slug>`,
`git ls-remote`, the reader verbs, the named rerun, `git ls-remote`.
The hook log shows each step entered: every phase below has its `upd` or `del` line twice, once for the killed attempt and once for the rerun.

| Phase | Slug, sleep pattern | Origin after the kill | `status` and `prune` | Origin after `fabric remove <slug> --remote` again |
| --- | --- | --- | --- | --- |
| before the archive tag push (the records branch push of an unpushed local tip) | `k1`, `upd refs/lym/heads/k1-weft` | `refs/heads/k1`, `refs/lym/heads/k1-weft` at the old tip `5a85cd3a` | both list `k1`, kind `remove`, `run lym fabric remove k1 --remote again` | `refs/heads/k1` (unlanded, kept) and `refs/lym/tags/archive/k1/5a85cd3a9de2` only — FAIL, see below |
| during the tag push | `k3`, `upd refs/lym/tags/archive/k3/` | both branches, no tag | both list `k3` | `refs/heads/k3` (unlanded, kept), `refs/lym/tags/archive/k3/47a4e6614dfe` |
| between the tag push and the records-branch delete | `k5`, `del refs/lym/heads/k5-weft` | both branches and the tag | both list `k5` | `refs/heads/k5` (unlanded, kept) and the tag |
| after the records-branch delete, before the code-branch delete | `k4` (no code commit, so landed), `del refs/heads/k4` | `refs/heads/k4` and the tag | both list `k4` | the tag only |

`cleanup` and `reconcile` named none of the slugs in any phase; `status` and `prune` did, and the rerun cleared the record each time. No stale lock blocked a rerun.

FAIL at phase 1 (`k1`). Before the removal the pair held one unpushed records commit, `a603ad1 weft sync`, adding `_lym/unpushed-rec.txt`.
The first run tagged it locally (`archive/k1/a603ad17c686`) and deleted the local branch; the child died in the branch push.
The rerun the pending record names archived origin's tip alone and deleted the records branch on origin.
`git ls-remote $O1 | grep k1` lists no ref at `a603ad1`; `git -C $O1 cat-file -t a603ad17c686` fails.
The commit is left under a local tag of this hub's records store that origin never got, and `status` and `prune` then list nothing.
Control, `k6`, the same unpushed records commit with no kill: origin holds `refs/lym/tags/archive/k6/f54d4ee791d3`, the local tip.

Killed discard, `k7` (a pushed pair with one unlanded code commit, `d2e6791c`): `batten cancel k7 --remove` from the prime,
sleep pattern `upd refs/lym/tags/archive/k7/d2e6791ce231` (the code tip's tag, the discard's own push), `kill -9` of the child.
After the kill origin held `refs/heads/k7` and the records tag; the local task branch was gone and the code store held the local tag `archive/k7/d2e6791ce231`.
`status` and `prune` listed `k7`, kind `discard`, `run the command that discarded the task branch again`.
`batten cancel k7 --remove` again answered `ok: true`. The hook log then holds `19:23:55 del refs/heads/k7` and no second `upd refs/lym/tags/archive/k7/d2e6791ce231`.
The hub log of that rerun: `kind=remote_branch_deleted target=k7 detail=origin`, `remote teardown done slug=k7 kind=remove`, `remote teardown done slug=k7 kind=discard`.
Origin now lists `refs/lym/tags/archive/k7/53b7f22df05d` alone: the task branch with unlanded work is deleted from origin and no archive tag of `d2e6791c` is there.
`git -C $O1 for-each-ref --contains d2e6791c…` lists nothing. FAIL: this is the charter's "task branch deleted from origin with no archive tag it can be recovered from".
The removal's landed check passed because the local tag the first run made holds the commit (`checkUnlandedRemoteTip` counts `--tags`, `internal/fabricengine/destroy.go:1046-1051`),
and the discard that follows found no origin branch left to archive. Both FAILs are finding N2.

Declined, not killed (`k2`, first attempt, where the kill hit this session's own shell instead of the child):
the hook declined the archive tag push after its sleep. The child logged
`the remote teardown of k2 failed at the archive of its records … way forward: push the archive tag with git -C $H1/origin-bare-weft push origin refs/tags/archive/k2/3513b6c3b9a2:refs/lym/tags/archive/k2/3513b6c3b9a2, then when origin still holds a branch of the pair, run lym fabric remove k2 --remote again`
and removed the pending record. `status`, `prune`, `cleanup`, `cleanup --remote` and `reconcile` then named nothing about `k2` while origin held both of its branches.
The two commands of that way forward worked when typed. A sandbox hub records no orch, so the notice reached the log alone; noted under Observations.

### P7 — push, sync and add killed mid-push (focus entry 5, second half)

Pair `p1` in `$H1`; each drive made one code commit and one records change, set the hook to sleep on one ref and to decline after, started the verb, and killed it with `kill -9` while the hook slept.

| Verb killed | Sleep on | Origin after the kill | `status`, `prune` | Next `fabric push` |
| --- | --- | --- | --- | --- |
| `fabric push` (the verb's own process) | `upd refs/lym/heads/p1-weft` (before the records push) | both branches at their old tips | list nothing | lands records then code |
| `fabric push` | `upd refs/heads/p1$` (records landed, code not) | records branch at the new tip, code branch one commit behind | list nothing | lands the code side |
| `fabric sync` (its detached push child) | `upd refs/heads/p1$` | as above | list nothing | lands the code side |
| `fabric sync` (child) | `upd refs/lym/heads/p1-weft` | both at old tips | list nothing | lands both |
| `fabric add p2` (its detached push child) | `upd refs/lym/heads/p2-weft` | no `p2` ref | list nothing; `pairs` shows `p2` `in_sync: true`; `reconcile` answers `already_healthy` | — |

No lock blocked a later verb, no ref left the namespace, and the pair's next push carried what the killed one left.
`fabric remove p2 --remote` on the never-pushed pair pushed `refs/lym/heads/p2-weft`, then `refs/lym/tags/archive/p2/39ce14ef955f`, then deleted the branch: all three in the namespace.
A killed push leaves no record, so no reader names the unpushed commits; the pair's next push is the only thing that carries them. WARN, noted under Observations.
`checkout`, `pull` and `reconcile` were not killed at a chosen phase: they fetch and write locally, a `pre-receive` hook cannot hold a fetch, and no other hold point was found without editing source. Named under Coverage.

Verdict: WARN.

### P8 — two clones of one origin at once (focus entry 8)

Pair 1 is `$H1` (A) and `$H1B` (B) on `$O1`. Pair 2 is `hub-849979879/origin-bare-LYMHUB` (A) and its clone under `hub-849979879/d/` (B) on `hub-849979879/origin-bare`.
Both pairs record the shortname `sbx`: `lym harden sandbox new` always writes it, and `lym fabric shortname sbx2` is refused on a hub that records one.
The hook was changed to sleep once per armed pattern, so the held push sleeps and the other clone's push of the same ref does not.

Prime records branch. Sequential control: A writes `_lym/conc-A-seq.txt` and pushes, then B writes `_lym/conc-B-seq.txt` and pushes; B's envelope carries `repo_advanced` and origin's `main-weft` tree holds both files.
Concurrent: A's `fabric push` sleeps in the hook on `upd refs/lym/heads/main-weft`, A's `lym` is stopped with `kill -STOP`, B pushes (`ok: true`), A is continued.
A's envelope: `commit_created`, `repo_advanced`, `branch_pushed`, `ok: true`. Hook log, pair 1: `7f2dc60e->11a4a626` (A, lost), `7f2dc60e->c55c7a4a` (B), `c55c7a4a->b0395f6c` (A, replayed).
Origin's tree holds `conc-A.txt` and `conc-B.txt`. Pair 2 gave the same shape (`43c7bff0->2c8df5e1`, `43c7bff0->30198780`, `30198780->56bd3da5`).
`git ls-remote` lists no ref outside `refs/heads/` and `refs/lym/` in pair 2, and in pair 1 only the leaked tag of P2.

Board. `lym board upsert` of a task is refused for this run session, which only the operator may do, so each clone wrote a note (`"kind":"note","labels":["bug"]`).
Sequential control: A upserts `seq-a` and syncs, B upserts `seq-b` and syncs; origin's `board.json` holds both.
Concurrent: A's `board sync` sleeps on `upd refs/lym/heads/main$` and is stopped, B upserts `conc-b` and syncs, A is continued.
Origin's board while A is held: `seq-a seq-b conc-b`; after A is released: `seq-a seq-b conc-b conc-a`. Same in pair 2. No board write was lost.

Same slug in both: P5.

A remote branch moved between A's read and A's delete. A and B both hold pair `m1` (B adopted it with `fabric add m1`).
A runs `fabric remove m1 --remote`; its child sleeps on `upd refs/lym/tags/archive/m1/` and is stopped; B pushes a records commit; A's child is continued.
Origin after: `refs/lym/heads/m1-weft` at B's commit, `refs/heads/m1`, and A's tag. A's hub log:
`remote teardown done slug=m1 kind=remove weft_kept="kept: origin's weft branch \"m1-weft\" moved during the removal, so it is another process's live branch"`,
and no `remote_branch_deleted` entry. B's next push landed. Same in pair 2.
Sequential control (`m2`): B pushes first, then A removes with `--remote`; origin holds `refs/lym/tags/archive/m2/<B's tip>` and no `m2-weft` branch, the documented outcome of `--remote`.

Verdict: OK on every claim driven, each with its sequential control and its second pair.

### P9 — reconcile, prune and cleanup against origin in each task state (focus entry 9)

Hub: pair 2's A (`$A`), its origin `$O`, and its second clone B. The origin reconcile was driven through `lym cleanup` (judge-only) and `lym cleanup --apply` (moving),
the verb that prints `ReconcileOrigin`'s report; `lym fabric reconcile` repairs local pairs and judges no origin branch.

States: `lv1` and `lv2` live and pushed; `ld1` pushed with no code commit (landed); `ab1` pushed, then both worktrees removed by hand with `git worktree remove --force` (abandoned);
`ow1` made and pushed from clone B alone (another clone's); `refs/heads/oo1` and `refs/lym/heads/oo1-weft` made on origin with `git update-ref` (origin-only);
`refs/heads/lv1-weft` made on origin by hand, a code branch with the short name of `lv1`'s records branch.
Then origin's `refs/lym/heads/lv2-weft`, `refs/heads/lv2` and the code branch `refs/heads/lv1-weft` were each advanced by one hand-made commit.

| Verb | Observed |
| --- | --- |
| `lym cleanup` | `lv2` code and `lv2-weft` records read `fast_forward`; `lv1-weft` records reads `level` with the namespaced head `72eda6f5`, not the code branch's `4487d21d`; tracking refs unchanged after the run (`44aac4d`, `7ada0e7`); `Entries: null` |
| `lym cleanup --apply` | `repo_advanced` for `lv2-weft` to `ac49533a` and `lv2` to `21f8d39c`; `lv1-weft` untouched; the records store holds no ref or object of the hand-made code branch |
| `fabric prune`, `fabric prune --apply` | no entry (the hand removal left no stale registration) |
| `fabric cleanup` | `ab1-weft` listed, not protected |
| `fabric cleanup --remote` (dry) | the same, and `warp_skipped_reason: … the open pull requests could not be listed … unrecognized remote URL shape` |
| `fabric cleanup --apply` | `ab1-weft` deleted locally, `archive_tag: archive/ab1/cfb807b48217`; origin then holds `refs/lym/tags/archive/ab1/cfb807b48217` and still `refs/lym/heads/ab1-weft` |
| `fabric cleanup --apply --remote` afterwards | nothing left to delete locally, so origin's `refs/lym/heads/ab1-weft` stays |

The control for the namespace read is `lv2-weft` against `lv1-weft`: the records branch whose namespaced ref moved reads `fast_forward`,
and the one whose same-name code branch moved reads `level`. An identity read would have swapped the two verdicts.
No verb tore down or judged `ow1`, `oo1` or `ab1`'s origin branches, and no archive tag was deleted.
`git ls-remote $O` lists no ref outside `refs/heads/` and `refs/lym/`, and the only `-weft` name under `refs/heads/` is the hand-made `lv1-weft`.

Not driven: the warp sweep of `cleanup --apply --remote`, which needs the open pull request list of a GitHub remote; and the ended-entry teardown of `ReconcileOrigin`,
which needs batten run records on the board that this run session may not write (a task upsert is refused). Both are named under Coverage.
Orphans no fabric verb names: an origin-only records branch (`oo1-weft`, and `ab1-weft` once its local branch is gone). Both layouts share that path, so it is noted, not raised.

Verdict: OK for the regions driven.

### P10 — joining, and the killed seed (focus entry 7)

Fresh bare code origins were made by hand under `$S/j/` (`git init --bare -b main`, one pushed commit), each with a logging `post-receive` hook that sleeps on one ref.
`$J` is `$S/j`.

Killed after the records branch landed (`k2`): `lym fabric clone --single-repo --shortname jx $J/k2 --into $J/h-k2`, hook sleeping after `refs/lym/heads/main-weft`, `kill -9` of the clone.
Origin then held `refs/heads/main` and `refs/lym/heads/main-weft` and no board: a mid-seed origin.

| Step | Observed |
| --- | --- |
| plain one-URL clone of the mid-seed origin into `$J/h-k2-join` | `weft $J/k2 has no recorded warp binding; supply the warp URL explicitly: lym fabric clone <weft-url> <warp-url>`; nothing made — WARN, see Observations |
| `clone --single-repo --shortname jx $J/k2 --into $J/h-k2-other` (another directory) | `ok`; origin's `refs/lym/heads/main-weft` replaced and `refs/lym/heads/main` seeded |
| `board sync` in the killed hub | `the board … has no upstream …, and origin holds another board at 21a24171…, which shares no history with this board at 7454ce1b…, so another clone seeded origin; way forward: remove the hub at … and run lym fabric clone <url> without --single-repo to join that hub` |
| `fabric push` in the killed hub's prime | class `not_fast_forward`, both local commits listed, the reset and merge advice |
| `lym fabric clone $J/k2 --reset` in the killed hub's directory (the join, with the removal done by `--reset`) | `ok`, `path_removed`, hub rebuilt; `board sync`, `fabric push` and `status` then answer ok |

The joined store's `git config --get-regexp 'remote\.|branch\.|lym\.'` holds the same settings as the seeding hub's:
both fetch refspecs, both push refspecs, `tagopt --no-tags`, `lym.primerecordsbranch main-weft`, and `branch.main` and `branch.main-weft` merging `refs/lym/heads/…`.

Killed after the board landed (`k3`), hook sleeping after `refs/lym/heads/main$`: origin held both namespaced branches and the records store held no `branch.*` setting.
`board sync` answered `… origin holds this hub's board at 8d8f0161…, so the seed landed and its clone stopped before it set the upstreams; way forward: set them with` four `git -C …/_bolt config branch.…` commands.
Each was typed exactly as printed and exited 0; `board sync`, `fabric push` and `status` then answered ok, and the store's settings equal a seeded hub's.

Other origin states:

| Origin | Clone | Observed |
| --- | --- | --- |
| holds `refs/heads/lym-bolt` (legacy board branch), never seeded | plain and `--single-repo` | both refused: `holds the branch lym-bolt, the layout of an earlier one-repository hub; way forward: delete lym-bolt on origin and remove .lym-anchor and _lym from the code branches, then clone again`; nothing made |
| a seeded origin (`k3`) that also holds `refs/heads/lym-bolt` | plain | the same refusal |
| default branch holds `.lym-warp` | `--single-repo` | `holds .lym-warp on its default branch main, so it is a records repository; way forward: make a two-repository hub with lym fabric clone <records-url> <code-url>`; nothing made |
| seeded (`k3`) | `--single-repo` again | `already holds a one-repository hub's board under refs/lym/; way forward: run the same clone without --single-repo to join it` |
| seeded (`k3`) | plain, into `$J/h-k3-join` | joined; the sorted config listing of its records store has the same digest as the seeding hub's (`7b276ac2e9`) |
| board of another clone with no common history; records branch at another tip | — | both are the killed `k2` hub after the second seed: the two rows above |

No refusal looped, no join took the identity layout, and every printed command worked when typed.
The seed killed before anything landed was not re-driven this round. Verdict: OK, with one WARN on the mid-seed join's wording.

### P11 — the layout markers and the namespace cache (focus entry 10)

Hub: pair 2's clone B. Its records store's settings were saved, then removed in groups, and after each group the prime ran `fabric push` (with a new records file) and `board sync`.

| Removed | `fabric push` | `board sync` | Origin |
| --- | --- | --- | --- |
| fetch refspecs and `lym.primeRecordsBranch` | refused: `holds the push refspec refs/heads/*:refs/lym/heads/*` | refused, same claim | no new ref under `refs/heads/` |
| the push refspecs too | refused: `holds the upstream setting branch.main-weft.merge refs/lym/heads/main-weft` | refused, same claim | no new ref under `refs/heads/` |
| nothing (settings restored, the listing equal to the saved one) | `ok: true` | `ok: true` | the records push landed under `refs/lym/heads/main-weft` |

Every pushing verb with `remote.origin.fetch` removed is P4.
A verb while another hub is cloned: six rounds of `fabric push` and `fabric status` in `$H1` ran while `lym fabric clone $J/k3 --into $J/h-conc` ran in the background; all answered ok,
and `fabric status` inside the hub being cloned answered ok on every read. The clone ended within the first rounds, so a read of its transient state was not shown to have happened.
Not driven: a long-lived process that outlives a rewiring. No fabric verb stays alive across verbs; the hub's reed server and board server do,
and starting either leaves a server the teardown check must find gone. Named under Coverage.

Verdict: OK for what was driven.

### P12 — owner archive tags read back from both stores (improvised, from the call-site count)

`remoteTagsUnder` and `readSideOwnerArchives` read `archiveTagNamespaceAt(dir)` since round 1, and `pushArchiveTag` pushes there, for both stores.
`ArchiveOwnerTips` runs only when another machine owns a worktree, which this session cannot stage, so the two tags it would push were made by hand in pair 2's A:
one commit origin lacks in each store (`git commit-tree HEAD^{tree} -p HEAD`), tagged `archive-owner/root/<sha12>`,
pushed with `git push origin refs/tags/<tag>:refs/lym/tags/<tag>`, the refspec `pushArchiveTag` builds.
Then `(cd $A/origin-bare && $L adopt --archives root)`, the read-only verb that calls `ReadOwnerArchives`.

Observed: four entries — `archive-owner/root/9c498c15db68` (the code commit) with side `code`, branch `main`, and again with side `records`, branch `main-weft`;
`archive-owner/root/dc6f39e372cd` (the records commit) with side `code`, branch `main`, and with side `records`.
Before the read each store held its own tag; after it each held both, `git -C $A/origin-bare cat-file -t dc6f39e3…` answers `commit`, and so does the code commit in the records store.
FAIL, finding G4.

## The call-site count (focus entry 2)

Commands, run from the worktree root, each piped through `grep -v '_test.go' | grep -v '^[^:]*:[0-9]*:\s*//'` to drop tests and whole-line comments,
over `internal/fabricengine/*.go internal/gitrepo/*.go internal/fabriccli/*.go`:

1. `grep -n 'refs/heads/\|refs/tags/\|refs/remotes/\|refs/lym'` — 71 lines.
2. `grep -n 'originNamespaceAt(\|archiveTagNamespaceAt(\|oneRepositoryNamespace()\|OriginRefNamesAt(\|RefNamespace{}'` — 61 lines.
3. `grep -n '\.Branch(\|\.Tag(\|\.LocalRef('` — 42 lines.

Total: 167 distinct lines, 174 with the lines two patterns share. `internal/fabriccli` holds none.
The table lists every line of pattern 1 and every call line of patterns 2 and 3; declaration lines and the lines inside `originNamespaceAt` itself are grouped in its row.

| Site | What it does | Verdict |
| --- | --- | --- |
| `gitrepo/refnamespace.go:10,11,24-56` | the two local prefixes and the mapping | correct: the only place the local prefixes are spelled for mapping |
| `gitrepo/push.go:154,165` | local source `refs/heads/<b>`, destination `ns.Branch` | correct: source is local, destination mapped |
| `gitrepo/push.go:212` | drops the local tracking ref | correct: local name |
| `gitrepo/push.go:308,352,385` | delete, leased delete, leased move by `ns.Branch` | correct: mapped |
| `gitrepo/pushdestination.go:74,117` | tracking ref and upstream setting from `ns.Branch` | correct: mapped, driven in P5 and P10 |
| `gitrepo/pushretry.go:87` | reads a rejected destination back through `ns.LocalRef` | correct |
| `gitrepo/pull.go:23,74` | prunes only refspecs whose destination is under `refs/remotes/` | correct: follows the store's own refspecs |
| `gitrepo/refreads.go:142,144` | local branch listing | correct: local names |
| `recordslayout.go:24-41,59-106,143-148` | the namespace constants, `originNamespaceAt`, `OriginRefNamesAt` | correct: the one spelling; claims driven in P4 and P11 |
| `recordslayout.go:91,96` | restore commands in the refusal | correct: typed in P4 |
| `recordslayout.go:172-174,182,194,195` | one-URL layout read: `ns.Branch("*")`, legacy `refs/heads/lym-bolt`, HEAD symref | correct: the legacy branch is a named constant read under identity by design |
| `recordsprobe.go:168` | probes `RecordsBranchName("main")` under the identity namespace | WRONG for a mid-seed one-repository origin: it reads a records ref under identity and misses `refs/lym/heads/main-weft`; observation O1 |
| `recordsprobe.go:225,226,240`, `resetsubpathcheck.go:87`, `clone.go:290,292` | branch probe by the namespace the clone passes | correct |
| `clone.go:823,828-831` | writes the records store's refspecs | correct: P1, P10 config listings |
| `clone.go:590,856` | local tracking refs | correct: local names |
| `destroy.go:1722,1756,1777,1800,1843,1844,1848,1857,1886,1896,1915,1919` | the seed's pushes, leases, rollback and refusal text | correct: the seed is one-repository by definition; P10 |
| `destroy.go:1736` | local records tip | correct |
| `pushclassified.go:140,182,203,207,258,731,735,909,994` | push job, board refusal, origin lookup, retries | correct: P4, P5, P8 |
| `pushclassified.go:557,582` | local tracking ref of a branch with no upstream | correct: P5 |
| `pull.go:329` | layout read before the pull | correct: P4 |
| `recordsgit.go:327`, `takeoverwrite.go:187`, `weftbranch.go:48,53`, `checkout.go:214,218,219`, `add.go:572,576` | records pushes and fetches by `ns.Branch` | correct; checkout and add driven in P4 |
| `add.go:498`, `remove.go:676`, `remoteteardown.go:495` | fetch `refs/heads/<task branch>` in the code store | correct: a code ref from the code store |
| `handover.go:275`, `destroy.go:1643` | identity namespace passed for the code branch of a pair | correct: both act on the code checkout only; not driven (hand-over needs a second machine) |
| `originreconcile.go:526,530,542,543,576,582,594,612` | origin heads by `ns.Branch("*")`, fetch into tracking refs | correct: P9, with the same-name control |
| `cleanupremotewarp.go:279,283,293,294,303,307,318` | origin branches and tags of a store by its namespace | correct as a read; `remoteTagRefs(weftRoot, …)` at line 100 now also returns code-side tags, part of G4 |
| `cleanupremotewarp.go:246,332` | local tag prefix after `LocalRef`; origin HEAD symref | correct |
| `cleanup.go:351,355` | layout read to find the prime's records branch | correct: P4 |
| `remoteleftover.go:144,149,151,179,183,197,247,251` | records branch and tag fetches by namespace, origin tip lookup | correct |
| `remoteleftover.go:279,283,284,296,301,303` | `remoteTagsUnder` by `archiveTagNamespaceAt` | WRONG: one prefix for both stores' tags, G4 |
| `ownerarchive.go:317,321` | fetches an owner tag by `archiveTagNamespaceAt` | WRONG: the same, G4 |
| `archive.go:159,368,372` | records branch push and lookup by the records store's namespace | correct: P6, P7 |
| `archive.go:230-258,268,279` | `archiveTagNamespaceAt`, the archive tag push | WRONG at 252-256: URL string compare falls back to identity, G1; shared prefix, G4 |
| `remoteteardown.go:188,190,205,207` | way-forward text `refs/tags/<tag>:<ns.Tag(tag)>` | correct as text; line 188 inherits G1 |
| `destroy.go:1071,1075,1602` | remote branch delete by the store's namespace, and its hand-delete advice | correct: P6, P8, P9 |
| `destroy.go:1016,1021,1048,1102,1103` | local reachability checks | local names, correct as names; the `--tags` at 1050 counts a local-only archive tag as a holder, which G3 rests on |
| `archive.go:267,334,352`, `destroy.go:2361`, `snapshotuncommitted.go:119,285`, `replay.go:309,310,547` | local tags and local prefixes | correct: local names |
| `beamline.go:125`, `boardweft.go:31`, `configchanges.go:156`, `parenttips.go:56,99`, `remove.go:485,882`, `recordsreads.go:134`, `remoteleftover.go:171,186`, `checkout.go:205`, `addadopt.go:61`, `worktreelist.go:55,108` | local branch and tracking-ref reads | correct: local names in either store |
| `seedlocalbares.go:92` | seeds a branch in a local bare code repository | correct: a code repository |

Wrong sites: `archive.go:252-256` (G1), `remoteleftover.go:279` with `ownerarchive.go:317` (G4), `recordsprobe.go:168` (O1).
No site was found that pushes, fetches or deletes a records branch under identity, or a code branch under the records namespace.

What the commands cannot see:

- A site that reads a records ref with no prefix and no namespace on the line. `Bolt.pullFirst` (`bolt.go:313-319`) is one: it calls `repo.Fetch()` and `repo.UpstreamSHA()`, and G2 lives there.
  Every `.Fetch()`, `Pull()`, `UpstreamSHA()`, `HasUnpushed()` and `HasUnpulled()` call is of this kind; they follow the store's own refspecs, which is right while the layout reads.
- A ref string built at run time from a variable holding a full ref name, and a prefix assembled from two constants on different lines.
- Refs built in tests and test kits, and callers that reach these functions through an interface or a function value, such as the `teardownHold` and the push notifier.
- The other modules: `battencli/readoptstop.go:172` reads `OriginRefNamesAt`, and `reportengine` reads local `archive/` tags of the records store, which G4's shared prefix also reaches.

## Focus entries

| Entry | Verdict | Probe |
| --- | --- | --- |
| 1 archive tag of a discarded task branch | FAIL | P1 holds; P2 respelled URL, G1 |
| 2 call-site count | FAIL | the count above: G1, G4, O1 |
| 3 records store with no fetch refspec | WARN | P4, G2 |
| 4 rejected first push | OK | P5 |
| 5 a teardown child that dies; verbs killed mid-push | FAIL | P6, G3; P7 WARN |
| 6 sandbox guard for pushes | OK | P3 |
| 7 joining and the killed seed | WARN | P10, O1 |
| 8 two clones at once | OK | P8, P5 |
| 9 reconcile, prune, cleanup per task state | OK for what a sandbox reaches | P9 |
| 10 namespace cache | OK for what was driven | P11, P4 |

## Observations the incremental frame does not hold

The review format admits a new finding in this round only at BLOCKING and only on a hunk round 1 changed.
The rubric says every finding is recorded, whatever its size. The entries below are real, smaller, or sit in files round 1 did not change, so they are recorded here and not in the frontmatter.

- **O1, LOW, consistency, `internal/fabricengine/recordsprobe.go:168` and the one-URL branch of `CloneHub` in `clone.go`.** CONFIRMED, P10.
  A plain one-URL clone of an origin that holds `refs/lym/heads/main-weft` and no board, which a killed seed leaves, answers
  `weft <url> has no recorded warp binding; supply the warp URL explicitly: lym fabric clone <weft-url> <warp-url>`.
  The origin is a code repository in mid-seed, and the advice describes a two-repository hub.
  `readOriginLayout` already lists the namespaced branches. Suggested fix: when `layout.heads` is non-empty and `layout.boardTip` is empty, refuse with
  "origin holds a one-repository seed that stopped before its board landed; way forward: run `lym fabric clone --single-repo <url>` to seed it", with a row in `TestCloneHub_OneURLRefusals`.
- **O2, LOW, consistency, `clone.go` (the `--into` handling).** CONFIRMED, P3 and P10.
  `lym fabric clone <url> --into <directory that does not exist>` answers `probe weft <url>: fork/exec /usr/bin/git: no such file or directory`.
  Git is there; the working directory is not. Suggested fix: create the `--into` directory, or refuse naming it, before the probe runs.
- **O3, LOW, design, `internal/fabricengine/remoteteardown.go:463`.** CONFIRMED, P6 (`k2`).
  A teardown child that fails a remote step without dying, here an archive tag push the remote declined, logs and mails its way forward and then removes the pending record.
  Origin still holds the pair's branches, and `status`, `prune`, `cleanup` and `reconcile` name nothing. A hub with no orch gets the notice in the log alone.
  Suggested fix: keep the pending record when `failedRemoteSteps` is non-empty, with the failed step's way forward in it, and clear it only after a job with no failed step.
- **O4, NIT, consistency, `pushclassified.go`.** One refused pair push raises the failed-push `count` by two, and the hub log holds two `push failed and was recorded` lines for one detached push (P4).
- **O5, noted.** A push killed outright leaves no record, so the commits it had not landed are named by no reader until the pair's next push (P7). No ref or lock is left wrong.
- **O6, noted, shared by both layouts.** `fabric pull` on a code branch that is only ahead of an unmoved origin answers `warp remote diverged and local warp has unpushed commits` (P4);
  an origin-only records branch is named by no fabric verb (P9).

## Focus departures

- "run each other mutating verb (add, pull, sync, checkout, remove, cleanup --apply, prune --apply, reconcile, clone, hand-over, replay)".
  `lym fabric` has no `hand-over` and no `replay` verb; both are engine functions that beam, dobby and the push retry call. The replay ran inside P4's control push and P8's lost races. Hand-over was not driven.
- "the same drive again in a second pair of hubs beside the first, with a different shortname".
  Every hub `lym harden sandbox new` makes records `sbx`, and `lym fabric shortname sbx2` is refused on a hub that records one. The second pair ran with the same shortname.
- "a pair of a sibling worktree's sandbox root if one exists". None exists on this machine; the case ran against paths under a sibling worktree's would-be root.
- "start the hub's reed server, rewire a store's fetch refspecs, delete the prime key by hand, run a push". The rewiring and the pushes ran from fresh processes (P11); no reed server was started.
- "Also kill … checkout, pull and reconcile at the phases the verb has". Not killed at a chosen phase (P7).
- "during it with a sleeping pre-receive hook in origin on refs/lym/tags/archive/*". Done, with the hook also declining after its sleep, so the killed child's orphaned `git push` could not land the step after the kill.
- "write a board entry in each". The entries were notes: a task upsert is refused for this run session.

## Coverage

Not driven, and why:

- The remote warp sweep of `lym fabric cleanup --apply --remote`: it needs a GitHub remote's open pull request list, which a local bare origin cannot give.
- `ReconcileOrigin`'s ended-entry and other-machine-owner teardowns: they need batten run records and owner entries on the board that this session's role may not write.
- `ArchiveOwnerTips`, hand-over, takeover and the code-tip reset (`handover.go:275`, `destroy.go:1643`): they need a second machine's ownership. G4's tags were pushed by hand for that reason.
- A long-lived process across a refspec rewiring, and a read of a hub in the middle of its clone.
- The seed killed before its first push landed, and `checkout`, `pull` and `reconcile` killed at a chosen phase.
- A real host: every origin was a local bare repository reached by path, so a host's refusal of `refs/lym/` pushes, its hidden refs and its hooks were not seen.

What the method failed to show about itself: the G4 read rests on hand-pushed tags, the concurrency drives held one process at one hook point each,
and a kill was simulated by `kill -9` of `lym` with the remote then declining the step in flight, which is one of the states a real crash can leave.

## Teardown check

```
$ lym harden sandbox clear
{"ended":[],"ok":true,"root":"/home/knatte/Code/loomyard-LYMHUB/harden-fabric-single-repo-verbs/.lym/harden-sandbox"}
$ lym harden sandbox list
{"hubs":[],"ok":true}
$ pgrep -af 'harden-sandbox' || true
(only this session's own shell, whose command line holds the pattern)
$ pgrep -af 'lym (reed|fabric|board)' || true
2095344 /home/knatte/go/bin/lym reed watchdog --hub-path /home/knatte/Code/scribe-LYMHUB --tmux tmux --shell bash
2097276 /home/knatte/go/bin/lym reed watchdog --hub-path /home/knatte/Code/loomyard-LYMHUB --tmux tmux --shell bash
$ ls .lym/harden-sandbox 2>/dev/null || true
hubs.json
hubs.json.lock
$ git -C . worktree list
/home/knatte/Code/loomyard-LYMHUB/loomyard                        4ff6aae9c [main]
/home/knatte/Code/loomyard-LYMHUB/harden-fabric-single-repo-verbs 8a8b8debb [harden-fabric-single-repo-verbs]
/home/knatte/Code/loomyard-LYMHUB/orch-autonomy-mode              118bee8ad [orch-autonomy-mode]
$ git -C . status --short
$ git -C . stash list
```

The two watchdogs are the real hubs' own, started by the installed `lym`, and name no sandbox path.
No hub is listed, no hub directory is left under the sandbox root, `.scratch/` is empty, and no source file was changed.
