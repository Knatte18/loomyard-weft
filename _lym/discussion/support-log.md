# Support log — single-repo-records-branches

## Interview

Autonomous run: no operator answered; every pick is an auto-pick in the question ledger.

### Turn 1 — the task and the board

The board body opens with `{"error":"invalid json ..."}` in place of its original text, then the operator's scope note:
the one-repository hub is for the operator's own use, the two-repository hub stays the standard,
and Discussion weighs a custom ref namespace such as `refs/lym/...` against plain branches.
The original body was read from the board history at `_bolt` commit 5a3ab0c956, before board sync b8e9afe900 replaced its opening.
Its three `To decide in Discussion` items are answered:
the records branch scheme by D1 to D4, what of f06acf39 stays by D5 to D7, and the layout probe's warnings by D2.
Title and brief ask for records on their own branches, wired as `_lym` like weft, the layout known only to fabric,
and a call on what of f06acf39 stays.
Touching entries: `two-repo-merge-probe-regression` (darn, running) reverts f06acf39's merge-state widening for two-repository hubs
and makes the layout probe quiet and cached; it leaves one-repository behaviour to this task.
`drop-compat-migration-code` records the operator's ruling that no migration code is wanted.
Project `single-repo-hub` groups the earlier Sketch's entries; its Sketch built records on each code branch.
No `Decision for single-repo-records-branches:` note exists.

### Turn 2 — exploration

Four subagents reported:

- Weft wiring: the weft is a plain `git clone` with default refspecs; fabric fetches and pushes it with explicit ref names
  (`resolveWeftBranch`, `PushWeft`, archive tag push, `cleanupremotewarp.go` reading `refs/tags/archive/*`);
  records branches are `<branch>-weft`; no code compares the two origin URLs or derives a GitHub slug from the weft remote;
  the board's `.lym-warp` names the warp URL.
- f06acf39 inventory: about half its volume is one-repository-only files (`clone_onerepository.go`, `recordspaths.go`,
  `recordscommit.go`, `recordsduringmerge.go`, `mergerecords.go`, `primepushguard.go`, `onerepository.go`) plus branches in shared files;
  general pieces are `ProtectedBranchRefusal`, `gitrepo`'s `remote:` lines and merge helper, and `PushOwedWayForward`.
  Two-repository widenings: the merge-state probe, deferred-push mail, protected refusal non-retryable everywhere,
  longer push errors, commit refusal order.
- Layout outside fabric: `lymcwd.OneRepository`, the duplicated commit-status seams in `battencli` and `loomcli`,
  `statuscommit`'s protected seam, `landingshed`'s abort-only wording; `operator_unpushed` and `protected` are reachable only in one-repository hubs.
- Git facts (git 2.53.0, local bare repos under the task's scratch directory): default clone and fetch skip `refs/lym/*`;
  mapped fetch and push refspecs work for bare `git push origin <branch>`, `git fetch origin`, `ls-remote`,
  `--force-with-lease` and both tag kinds;
  `git push origin --delete <branch>` fails under the mapping, the full ref name works
  (superseded in round 1: `gitrepo` pushes to the URL, where a full-name delete of a missing ref exits 0; see D4);
  a bare `git push origin` with the tag push refspec also pushes every local tag;
  `git clone --mirror` copies the namespace.
  GitHub: no official page lists accepted namespaces; `refs/pull/` is hidden and read-only;
  a 2023 git mailing-list thread shows GitHub reporting a `refs/notes/commits` push as updated;
  rulesets and branch protection target branches and tags only.

### Turn 3 — design

Recommended and auto-picked: one origin with two local stores, the records store wired as weft with its refs mapped under `refs/lym/`.
This keeps a one-repository hub's local behaviour identical to a two-repository hub's, so nearly all of f06acf39's machinery has no case left.
The layout lives in the records store's fetch refspec, read and cached by fabric alone.

### Turn 4 — lens review

All six forks returned a report.

- existing-paths 1, bound-breaker 1 (BLOCKING, merged): `gitrepo` pushes to a URL with explicit `refs/heads/` destinations and lease refs,
  bypassing the push refspec — fixed: D2 passes the namespace to `gitrepo`'s push, lease and result matching.
- existing-paths 2 (BLOCKING): upstream `branch.<b>.merge` written untranslated — fixed in D2, with a criterion.
- existing-paths 3, walkthrough 1, existing-paths 4 (merged): `ls-remote --heads`/`--tags` and prefix-stripped output miss the namespace
  and read code branches — fixed: D2 selects by namespaced pattern, maps output back, and forbids reading origin's `refs/heads/`
  from the records store; criterion added.
- existing-paths 5: push result lines matched only against `refs/heads/` — fixed in D2's list.
- existing-paths 6: remote-teardown hint survives — no change needed.
- existing-paths 7, bound-breaker 7: remote `HEAD` read — fixed: a records-store `HEAD` read answers the board branch.
- interactions 1, bound-breaker 2 (merged): cache of a missing store fails open — fixed: a missing store is not cached,
  an unreadable store is an error; criterion added.
- interactions 2: partial seed push leaves a half hub on origin — fixed: records branch first, board branch (the detection key) last,
  and a leftover namespaced records branch without a board is replaced; criterion added.
- walkthrough 2: refused push inside `CloneAndWire` leaves the hub intact — fixed: the seed push runs before `CloneAndWire`
  (superseded in round 1: the seed push takes the board push's step inside `CloneAndWire` and tears the hub down itself).
- interactions 3, bound-breaker 4 (merged): re-cloning an f06acf39 origin passes every refusal — fixed: D3 refuses an origin holding
  `lym-bolt` or a root `.lym-anchor` and names the cleanup; D10 reworded; criterion added
  (narrowed in round 1: the `.lym-anchor` and `_lym` arm applies with `--single-repo` only).
- interactions 4, board-scope 2 (merged): D7 criterion targeted the code worktree — fixed: records worktree;
  D7 now defers to the peer's revert when it lands first.
- interactions 5: kept protected-refusal widening unstated in D7 — fixed.
- interactions 6: bare push sends every tag — fixed: D2 forbids a bare fabric push, D9 lists the hazard.
- walkthrough 3, bound-breaker 3 (merged): deferred-mail revert described wrongly — fixed: no deferred record is mailed;
  criterion now names a record with commits.
- walkthrough 4, bound-breaker 5 (merged): records store as a plain clone fetches code refs — fixed: initialised with refspecs before
  its first fetch; criterion added.
- walkthrough 5, bound-breaker 6 (merged): `--single-repo` over a two-repository hub's code origin — fixed as a D9 hazard;
  a refusal was rejected since nothing on that origin marks it.
- board-scope 1 (decision): peer run overlap — kept under Open risks for the parent, now naming the narrowing option.
- board-scope 3: stale project title — added to Open risks for the parent.
- leanness: Scope Out bullet on ordinary branches — fixed, cut.
- leanness: GitHub-push open risk repeats D1 and D3 — fixed, cut.
- leanness: board-body assumption duplicates Open risks — fixed, assumption cut.
- leanness: D6 last bullet repeats `PATTERN-test-only-code` — fixed: moved into the Constraints line.
- leanness: support-log pointers in D1 and D4 — fixed, cut.
- leanness: Goal's last sentence previews D5 and D6 — rejected: it states the f06acf39 call the brief asks for.
- leanness: D10's `Review:` criterion on code absence — fixed: replaced by a test of D3's f06acf39-origin refusal.

## Rejected alternatives

- **Keep f06acf39's records on each code branch.**
  Records ride every merge, pull request, replay and the operator's own pushes and rebases; it needs a records strip at every merge,
  a guarded prime push, two new failed-push classes and layout seams outside fabric, and a protected prime branch strands records.
  The title and brief ask for records on their own branches.
- **Records on ordinary branches under a prefix such as `lym/records/<slug>`.**
  They appear in GitHub's branch list, branch dropdown and "recently pushed branches" banners, and every default clone fetches them:
  the clutter the operator's scope note names.
  Their one gain, branch protection by pattern, is not wanted for records.
- **Records branches in the code store itself, one local store.**
  Weft worktrees would share the code store's refs, so the code store's default fetch would need narrowing, `git branch` would list records,
  and the junction wiring would differ from two-repository hubs;
  two stores keep the local geometry the same but for the board's branch.
- **A separate config key or hub file recording the layout.**
  It could disagree with the refspec git obeys; the refspec is the one source.
- **Relying on the configured push refspec alone, with no explicit namespace.**
  `gitrepo` pushes to a URL with explicit destinations, which bypasses `remote.origin.push`.
- **An atomic push of the seed refs.**
  Ordering the detection key last needs no host capability; host support for `--atomic` was not checked.
- **Refusing `--single-repo` over a two-repository hub's code origin.**
  Nothing on that origin marks it; the hazard is documented instead.
- **Reading the prime's records branch from the code store's `origin/HEAD` or the records worktree's branch.**
  `git remote set-head` moves `origin/HEAD`, and `lym fabric checkout` moves the records worktree,
  so either could turn the prime's records line into an orphan the sweep deletes;
  a config key only the clone writes cannot move (D1).
  It records the prime's branch, not the layout, so it does not compete with the refspec.
- **Converting f06acf39-layout hubs.**
  The layout is hours old and the operator rejects migration code; a refusal naming the cleanup suffices.

## Review rounds

### Round 1

Every finding fixed in the decision record:
the seed push takes the two-repository board push's step inside `CloneAndWire`, pushes records then board under leases,
and covers every failed push (F1, F5, F10);
a clone whose board push loses its lease returns the records branch it replaced (self-check);
the `.lym-anchor` and `_lym` refusal arm applies with `--single-repo` only, so a two-repository weft still clones (F2);
teardown checks the full ref name with `ls-remote` before deleting, in both layouts,
and D4's delete evidence is the URL push `gitrepo` runs (F3);
the board branch is the fixed name `main` (F4);
a `--single-repo` rerun replaces a leftover records branch (F6);
the lost-body Open risks line is settled from board history (F7).

### Round 2

F14 fixed in the decision record:
D1 names the board worktree's branch as the one local difference from a two-repository hub;
both clone paths hand `ensureBoardWorktree` the board name;
`--single-repo` makes its records store by `git init` and fetches nothing, so the fresh bind cannot adopt a leftover records branch;
the prime's records branch is recorded in the records store's config key `lym.primeRecordsBranch`,
which `primaryWeftBranch` reads in a one-repository hub, and a missing key's refusal names how to set it;
a criterion runs the clone, second-machine clone and sweep over a `master` default branch.
The self-check's findings were fixed:
the checkout claim names the key, the Goal, D3's rationale and this log's geometry claim name the board-branch difference,
and D5 separates the removed `lym.primeBranch` from the new key.

## Question ledger

- Q1 (auto-pick): custom namespace `refs/lym/` or prefixed ordinary branches? — `refs/lym/heads/` and `refs/lym/tags/`;
  hidden from GitHub's branch and tag lists and default clones, tested working with mapped refspecs.
- Q2 (auto-pick): where is the layout recorded? — the records store's `remote.origin.fetch` refspec, read by fabric.
- Q3 (auto-pick): which f06acf39 pieces stay? — `ProtectedBranchRefusal` and its non-retryable classification,
  `gitrepo`'s `remote:` lines, and any general helper a remaining production path calls; the rest goes (D5, D6).
- Q4 (auto-pick): convert existing f06acf39-layout hubs? — no; refuse their origin with the cleanup named, then re-clone (D3, D10).
- Q5 (auto-pick): revert the two-repository widenings? — yes, all three in D7, matching `two-repo-merge-probe-regression`.
- Q6 (resolved by research): the board body's lost text — read from the board history at `_bolt` commit 5a3ab0c956;
  its three `To decide in Discussion` items are answered by D1 to D4, D5 to D7 and D2.
- Q7 (open, parent): order against the running peer `two-repo-merge-probe-regression`, which edits files D5 deletes.
- Q8 (resolved by research): does GitHub accept pushes to `refs/lym/*`? — evidence says yes for non-reserved namespaces;
  D3 refuses the clone on a host that rejects it.
- Q9 (auto-pick): seed push order — records branch first, board branch last (D3).
- Q10 (open, parent): retitle project `single-repo-hub`.
- Q11 (auto-pick): where does a one-repository hub record its prime's records branch? —
  the records store's git config key `lym.primeRecordsBranch`, written by the clone (D1).
