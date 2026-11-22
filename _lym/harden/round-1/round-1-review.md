---
verdict: BLOCKING
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/fabriccli/roleguard.go:174
    summary: "The sandboxed-role guard passes any `lym fabric push` that carries --weft-path or --warp-path, whatever paths they name, so a task session's push run from the real hub pushes another hub's stores"
    fix: "In sandboxedRoleRefusal, let a push with injected paths pass only when every injected path lies under inOwnSandboxRoot for the strand name, or is the strand's own task worktree or its weft sibling (the worktree named by name.Slug in the hub whose shortname matches, as inOwnSandboxRoot already resolves it); refuse it otherwise with the existing refusal text. Add rows to the refusal test in internal/fabriccli/cli_refusals_test.go: a push whose injected paths name a pair outside the sandbox root and outside the strand's own worktree is refused, the strand's own pair passes, and a pair under its own sandbox root passes."
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/fabricengine/archive.go:224
    summary: "A discarded task branch's archive tag lands in plain refs/tags/ on a one-repository origin: pushArchiveTag reads the namespace of the code store, which is identity, so `lym batten cancel <slug> --remove` pushed refs/tags/archive/t4/3e9e8037adc3"
    fix: "In pushArchiveTag (and archiveTaskTip, which calls it with the code worktree), push a task-side archive tag of a one-repository hub under the records namespace: when the hub's records store (RecordsRepoRoot) reads as the one-repository namespace through originNamespaceAt and its origin URL is the code store's origin URL, use that namespace's Tag(tag) as the destination instead of the code store's identity namespace. Make the origin-side archive check that gates the origin delete in deleteArchivedTaskBranch (dirtyArchivedOnRemote) read the same namespaced tag ref. Add a regression test over hubforge's one-repository builder in internal/fabricengine/taskbranchdiscard_integration_test.go: after DiscardTaskBranch of a branch with an unlanded commit, ls-remote of origin lists refs/lym/tags/archive/<slug>/<sha12> and no ref under refs/tags/archive/."
  - id: F3
    severity: MEDIUM
    class: consistency
    location: internal/fabricengine/pushclassified.go:573
    summary: "unpushedCommits returns nil for a branch with no upstream, so a rejected first push's way forward says \"local commits: none\" and advises `git reset --keep`, which drops the commit it failed to list"
    fix: "In unpushedCommits, read origin's tip the way failRejection does: on gitrepo.ErrNoUpstream fall back to ResolveSHA(\"refs/remotes/origin/\" + j.branch), and list CommitsNotIn(head, that tip). Add a row to the not-fast-forward test of the push classifier: a records branch with no upstream whose first push is rejected names its own local commit in the way forward."
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/fabricengine/pushclassified.go:617
    summary: "A pair's records push in a one-repository records store with no fetch refspec fails with the raw \"gitrepo: resolve upstream refs/remotes/origin/lym/heads/<b>: reference not found\", naming no way forward, and the detached push's failure never reaches failed_pushes"
    fix: "At the start of the push job (before checkOrigin's UpstreamSHA read), call originNamespaceAt(j.path) and return its error, so the records-store refusal with its restore commands is what the push reports. Record that refusal as a failed push of class other with its way forward, so `lym fabric status` lists it after a detached push. Add a test over hubforge's one-repository builder: unset remote.origin.fetch in the records store, run the pair push, and assert the error names `git -C <store> config --replace-all remote.origin.fetch` and that the failed-push record exists."
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/fabricengine/pull.go:326
    summary: "`lym fabric pull` in a one-repository records store with no fetch refspec reports ok with weft_pulled true and moves nothing, although origin's records branch is ahead"
    fix: "In pullWeftArm, before weftHasUpstream, call originNamespaceAt(f.weftPath) and return its error as the pull's error rather than warning past it, so the pull fails with the records-store refusal and its restore commands. Add a test over hubforge's one-repository builder: unset remote.origin.fetch in the records store, advance origin's refs/lym/heads/<branch>, run Pull, and assert it fails naming the restore commands and leaves weft_pulled false."
  - id: F6
    severity: MEDIUM
    class: design
    location: internal/fabricengine/remoteteardown.go#RunTeardownChild
    summary: "A `lym fabric remove --remote` whose teardown child is killed with -9 during its archive tag push leaves the pair's code branch and records branch on origin, and status, prune, cleanup and reconcile all report nothing; only rerunning the same remove clears them"
    fix: "Write a pending-teardown record for the slug in the hub (beside the teardown lock under _bolt/.lym/teardown/) before the job is handed to the child, and delete it when the child reports done. Make `lym fabric status` and `lym fabric prune` list a pending-teardown record whose lock no process holds, naming `lym fabric remove <slug> --remote` as its way forward. Add a test that writes the record with no lock holder and asserts status names the way forward, and one that a finished teardown leaves no record."
  - id: F7
    severity: LOW
    class: consistency
    location: internal/fabriccli/roleguard.go:187
    summary: "The task-session refusal names only a hubforge fixture and a FAILED stop as ways forward, not the sandbox hub that lets the verb run"
    fix: "Add to the refusal text: \"or run it in a hub made with `lym harden sandbox new` under <worktree>/.lym/harden-sandbox\", built from sandboxRootRel; update the refusal-text assertion in internal/fabriccli/cli_refusals_test.go."
---

### [MEDIUM] The push bypass flag carries any task session past the sandbox guard

**Location:** `internal/fabriccli/roleguard.go:174`

**Issue:** `sandboxedRoleRefusal` returns "" for every `push` that carries `--weft-path` or `--warp-path`, before it reads any path.
From this worktree's root (the real hub, outside the sandbox root), `.dev-bin/lym fabric push --weft-path $H/t1-weft --warp-path $H/t1` pushed hub A's code branch `t1` to its origin (probe 3).
The same call with the real hub's pair paths pushes the real hub, so the sandbox exception does not keep the push verb refused against the real hub.
CONFIRMED.

### [BLOCKING] A discarded task branch's archive tag lands in plain `refs/tags/` on a one-repository origin

**Location:** `internal/fabricengine/archive.go:224`

**Issue:** `archiveTaskTip` tags the code tip in the code store and `pushArchiveTag` reads `originNamespaceAt(repoDir)` there.
The code store keeps git's default refspecs, so it reads as identity and the tag goes to `refs/tags/archive/<slug>/<sha12>`.
Probe 7: `lym batten cancel t4 --remove` on a one-repository hub; origin then holds `refs/tags/archive/t4/3e9e8037adc3` beside the records tag `refs/lym/tags/archive/t4/b11aec723a31`.
A plain clone of that origin (probe 8, hub B) fetched the leaked tag into its code store.
The decision record states that the archive tag of a discarded task branch lands under `refs/lym/tags/`, never plain `refs/tags/`.
This is the plain-`refs/tags/` tag the sandbox trial saw: verb `batten cancel --remove` (and `batten cleanup --apply` through `discardTaskBranchIfPresent`), store the code store.
CONFIRMED.

### [MEDIUM] A rejected first push names no local commits, then advises a reset that drops one

**Location:** `internal/fabricengine/pushclassified.go:573`

**Issue:** Probe 10: the same slug added in clones A and B.
B's records branch never gained an upstream, since its first push was rejected.
`lym fabric push` in B refuses with "local commits: none" and advises `git reset --keep origin/dup-weft`, "which drops the local commits", while `git rev-list --count origin/dup-weft..HEAD` is 1.
`failRejection` falls back to `refs/remotes/origin/<branch>` on `ErrNoUpstream`; `unpushedCommits` does not, and answers nil.
A reader who trusts "none" loses commit 8916cce.
CONFIRMED.

### [MEDIUM] A pair push with no namespaced fetch refspec fails with no way forward

**Location:** `internal/fabricengine/pushclassified.go:617`

**Issue:** Probe 12: with `remote.origin.fetch` removed from a one-repository records store, `lym board sync` refuses with the designed records-store message and its restore commands.
The pair push in a task worktree instead fails with "gitrepo: resolve upstream refs/remotes/origin/lym/heads/k1-weft: reference not found", which names no way forward.
`checkOrigin` reads `UpstreamSHA` before anything reads the layout, so `recordsStoreClaim`'s refusal never runs.
After `lym fabric sync`, the detached push failed with the same text in the hub log, and `lym fabric status` reported `failed_pushes: []`.
CONFIRMED.

### [MEDIUM] Pull reports a weft pull it did not make when the layout is broken

**Location:** `internal/fabricengine/pull.go:326`

**Issue:** Probe 12: with the records store's fetch refspecs removed and origin's `refs/lym/heads/k1-weft` advanced to 68a4cd8, `lym fabric pull` answered `ok: true`, `weft_pulled: true`, with no mutation, and the local `k1-weft` stayed at 2423f42.
`pullWeftArm` warns past an upstream read error and never reads the layout, so a records store that would push into `refs/lym/` pulls from nowhere and calls it done.
Control: after restoring the refspecs with the commands `board sync` printed, the same pull recorded `repo_advanced k1-weft 68a4cd8`.
CONFIRMED.

### [MEDIUM] A killed teardown child leaves origin branches that no reader verb names

**Location:** `internal/fabricengine/remoteteardown.go#RunTeardownChild`

**Issue:** Probe 15: a pre-receive hook in origin stalled the archive tag push of `lym fabric remove t8 --remote`, and its `remote-teardown` child was killed with `kill -9`.
The local pair was gone, while origin kept `refs/heads/t8` and `refs/lym/heads/t8-weft` beside the pushed archive tag.
`fabric status`, `prune`, `cleanup` and `reconcile` from the prime all reported nothing about them.
`warnWhenEnded` in `internal/fabriccli/teardownchild.go` handles only SIGINT and SIGTERM, so a SIGKILL leaves no notice either.
Rerunning `lym fabric remove t8 --remote` cleared both branches, but nothing tells a reader to run it.
CONFIRMED.

### [LOW] The task-session refusal omits the sandbox hub as a way forward

**Location:** `internal/fabriccli/roleguard.go:187`

**Issue:** Probe 2's refusal names "build a scratch hub as a hubforge test fixture instead" and a FAILED stop.
It does not name `lym harden sandbox new`, the way a hardener round makes the verb run, although the guard admits it.

## Focus entries

| Entry | Result | Probes |
| --- | --- | --- |
| Sandbox exception | FAIL (F1, F7) | 2, 3 |
| Joining | OK for a seeded origin | 8 |
| Namespace leaks | FAIL (F2) | 4, 5, 7, 13 |
| Archive tags | FAIL (F2); records tags OK | 5, 7, 13, 14 |
| Two clones at once | WARN (F3); board and prime records OK | 9, 10, 16 |
| Remote delete | OK | 13, 14 |
| A pair verb killed mid-operation | FAIL (F6) for `remove` | 15 |
| A killed seed and its way forward | OK, 0e4782373 holds | 11 |
| Namespace cache and layout read | FAIL (F4, F5) | 12 |
| Reconcile, prune and cleanup against origin | OK on the states driven | 4, 5, 15 |

## Probe log

Variables used below:
`L=.dev-bin/lym` (the lym `lym harden sandbox new` printed),
`S=.lym/harden-sandbox/hub-507258856`, `H=$S/origin-bare-LYMHUB` (hub A), `O=$S/origin-bare` (the bare origin),
`HB=$S/b/origin-bare-LYMHUB` (hub B), `HK=.lym/harden-sandbox/ks1/hub/code-LYMHUB`.

1. `lym harden sandbox new --single-repo` → hub A at `$H`, `.dev-bin/lym`, `single_repo: true`.
   Origin holds `refs/heads/main`, `refs/lym/heads/main` (board), `refs/lym/heads/main-weft` (prime records).
   Records store config: fetch `+refs/lym/heads/*:refs/remotes/origin/*` and `+refs/lym/tags/*:refs/tags/*`,
   push `refs/heads/*:refs/lym/heads/*` and `refs/tags/*:refs/lym/tags/*`, `tagopt --no-tags`,
   `lym.primerecordsbranch main-weft`, `branch.main.merge refs/lym/heads/main`, `branch.main-weft.merge refs/lym/heads/main-weft`.
   Code store keeps git's default refspec. Seed layout OK.
2. Sandbox guard, real hub: from this worktree, `$L fabric remove nonexistent-xyz-probe` →
   refused, "lym fabric remove is refused from the task session ... outside a hub under its worktree's .lym/harden-sandbox".
   Same verb from `$H/origin-bare` → passes the guard, fails on "pair not found". Guard keys on the resolved hub path: OK.
3. Sandbox guard, push bypass: from this worktree's root (the real hub),
   `$L fabric push --weft-path /nonexistent/weft --warp-path /nonexistent/warp` → `ok: true` (passed the guard).
   Then `$L fabric push --weft-path $H/t1-weft --warp-path $H/t1` from the real worktree's cwd →
   `branch_pushed t1 side=code` and origin's `refs/heads/t1` moved to the new commit.
   The guard never checks the injected paths, so a task session's push reaches any hub's stores (F1).
4. `$L fabric add t1` in hub A → origin gains `refs/heads/t1` and `refs/lym/heads/t1-weft`; `branch.t1-weft.merge refs/lym/heads/t1-weft`.
   Then `push`, `sync`, `pull`, `status`, `pairs`, `reconcile`, `prune`, `cleanup` in `$H/t1`:
   every ref on origin stays in its namespace (records under `refs/lym/heads/`, code under `refs/heads/`). OK.
5. `$L fabric remove t1` with an unpushed real commit on t1 → code branch kept with the reason, records archive tag pushed to
   `refs/lym/tags/archive/t1/a9e1f97b8e83`, no plain `refs/tags/` ref on origin. OK.
   `cleanup --apply --remote` → nothing to do; the warp half skipped because the local bare URL is no GitHub URL (expected).
6. Pairs t2 and t3 with `--allow-empty` commits: `lym batten cancel t2 --remove` and `$L fabric remove t3 --remote --force`
   deleted the task branch locally and on origin, leaving the commits unreachable.
   Read `checkUnlandedWork` and `checkUnlandedRemoteTip` (`destroy.go`): an empty commit merges into the parent with no change,
   so it counts as landed by design. Not a defect; the later drives use real file changes.
7. Archive tag of a discarded task branch: `$L fabric add t4`, commit a real file, push the code branch,
   then `$L batten cancel t4 --remove` from the prime (it reaches `DiscardTaskBranch`).
   The remove's remote half kept the code branch ("1 commit(s) on the remote branch are reachable from no other ref"),
   and the discard's remote half logged `tag_pushed target=origin-bare detail=archive/t4/3e9e8037adc3`.
   `git -C $O for-each-ref` then shows `refs/tags/archive/t4/3e9e8037adc3` on origin: an archive tag in plain `refs/tags/` (F2).
   This is the plain-`refs/tags/` archive tag the decision record names: the code store's discard, through `archiveTaskTip`.
8. Joining: `$L fabric clone $O --into $S/b` (no `--single-repo`) from inside the sandbox root → hub B.
   B's records store matches A's: both fetch refspecs, both push refspecs, `tagopt --no-tags`, `lym.primerecordsbranch main-weft`,
   `branch.main.merge refs/lym/heads/main`; its records tags are `archive/t1..t4` from `refs/lym/tags/`.
   `board list`, `fabric status`, `fabric pairs` in B all answer. Layout OK.
   B's code store holds `refs/tags/archive/t4/3e9e8037adc3`, fetched by its plain clone from F2's leaked tag.
9. Board writes from two clones: `board upsert` of a note in A and in B, then `board sync` in A (origin `refs/lym/heads/main` → 7b12630)
   and in B (→ 33fb0a2, a linear commit on top). B's list shows both notes (from-a B1, from-b B2). No write lost: OK.
   A's `board sync` with nothing pending does not fetch, so A keeps showing only its own note until its next write.
   That is board behaviour, outside this campaign's modules; not raised.
10. Same slug in two clones: `$L fabric add dup` in A, then in B. Origin keeps A's `refs/lym/heads/dup-weft 64f0576`;
    B's detached push is rejected. `$L fabric push` in `$HB/dup` refuses:
    "origin/dup-weft holds commits this branch lacks (local commits: none); to keep origin's side, run `git reset --keep origin/dup-weft` ..., which drops the local commits".
    `git rev-list --count origin/dup-weft..HEAD` in B's `dup-weft` is 1 (8916cce), and `git config --get-regexp 'branch\.dup'` is empty:
    B's branch has no upstream since its first push never landed.
    `unpushedCommits` (`pushclassified.go:573`) reads only `UpstreamSHA` and returns nil on `ErrNoUpstream`,
    so the way forward says "none" while its reset drops a commit (F3).
    `fabric status` in B lists the same record under `failed_pushes` once the detached push has finished.
11. Killed seed, re-drive of 0e4782373 (`.scratch/killseed.sh <dir> <ref>`: a fresh code bare under the sandbox root,
    a `post-receive` hook that sleeps on `<ref>`, `lym fabric clone --single-repo --shortname sd`, `kill -9` of the clone once the hook runs).
    - `ks1`, stalled on `refs/lym/heads/main`: origin holds `refs/lym/heads/main` and `main-weft`, the hub has no upstreams.
      `lym board sync` → "origin holds this hub's board ..., so the seed landed and its clone stopped before it set the upstreams",
      naming four `git -C <bolt> config branch.*` commands. Typed exactly, then `board sync`, `board upsert` + `board sync`,
      `fabric add k1`, `fabric pairs` → all `ok: true`; origin gains `refs/lym/heads/k1-weft`. OK.
    - `ks2`, stalled on `refs/lym/heads/main-weft`: origin holds only `main-weft`.
      `board sync` → "origin holds no board under refs/lym/, so it was never seeded onto origin; way forward: ... `--reset`".
      The `--reset` clone → `ok: true`, origin's `main-weft` replaced and `main` seeded. OK.
    - `ks3`, stalled on `refs/lym/heads/main`, then the `--single-repo --reset` clone instead of the printed commands →
      refused "already holds a one-repository hub's board ...; way forward: run the same clone without --single-repo";
      that join clone with `--reset` → `ok: true`, upstreams `refs/lym/heads/main` and `main-weft` set. No loop. OK.
12. Layout read with the records store's fetch refspecs removed (`git -C $HK/code-weft config --unset-all remote.origin.fetch` in `ks1`):
    - `board sync` in the prime refuses with the designed records-store message and restore commands. OK.
    - `fabric push` in `$HK/k1` → "push records side ...: gitrepo: resolve upstream refs/remotes/origin/lym/heads/k1-weft: reference not found",
      no way forward; `fabric sync` spawns a detached push that fails with the same text in the hub log,
      and `fabric status` then reports `failed_pushes: []` (F4).
    - Origin's `refs/lym/heads/k1-weft` advanced by hand to 68a4cd8, then `fabric pull` in `$HK/k1` →
      `ok: true`, `weft_pulled: true`, no mutation; the local `k1-weft` stays at 2423f42 (F5).
      Control: restore the refspecs with the commands `board sync` printed, `fabric pull` again → `repo_advanced k1-weft 68a4cd8`.
13. Remote delete, same short name in both namespaces: `git -C $H/origin-bare push origin main:refs/heads/t5-weft`, `$L fabric add t5`,
    then `$L fabric remove t5 --remote`. The teardown child logged `remote_branch_deleted t5-weft` and `t5`;
    origin then holds `refs/heads/t5-weft` (the code branch, kept) and `refs/lym/tags/archive/t5/91401b083857`,
    and no `refs/lym/heads/t5-weft`. OK.
14. Remote delete of a records branch deleted by hand: `$L fabric add t7`, `fabric push` (origin lists `refs/heads/t7` and `refs/lym/heads/t7-weft`),
    `git -C $O update-ref -d refs/lym/heads/t7-weft`, then `$L fabric remove t7 --remote`.
    The removal's own push put `t7-weft` back before its teardown child archived it and deleted it,
    so the logged `remote_branch_deleted t7-weft` is a real deletion; `DeleteRemoteBranch` lists the ref before it deletes (`push.go:309`).
    Origin ends with only `refs/lym/tags/archive/t7/568e10174c28`. OK.
    (A first attempt on t6 raced the add's detached push and is discounted.)
15. Killed teardown child: `$L fabric add t8`, `fabric push`, a `pre-receive` hook in `$O` that sleeps on `refs/lym/tags/archive/*`,
    `$L fabric remove t8 --remote` in the background, `kill -9` of `lym fabric remote-teardown --slug t8` once the hook ran.
    The local pair is gone; origin keeps `refs/heads/t8`, `refs/lym/heads/t8-weft` and the archive tag.
    `fabric status`, `prune`, `cleanup`, `reconcile` from `$H/origin-bare` → nothing about t8 (F6).
    The stale `_bolt/.lym/teardown/t8.lock` file does not block: `$L fabric remove t8 --remote` again → `ok: true`, and its child deleted both branches.
16. Prime records branch from two clones: a new file in each prime's records worktree, `$L fabric push` in A (origin `main-weft` → cc200e3),
    then in B → `repo_advanced` and `branch_pushed`, origin `main-weft` → 8b212a2 with both files. No write lost. OK.
17. Teardown check:
    `lym harden sandbox clear` → `{"ended":[],"ok":true,...}`; `lym harden sandbox list` → `{"hubs":[],"ok":true}`.
    `pgrep -af 'harden-sandbox'` → only the check's own shell.
    `pgrep -af 'lym (reed|fabric|board)'` → the real hubs' reed watchdogs and a push of the `later-transition-push-wait` worktree; nothing under the sandbox root.
    `ls .lym/harden-sandbox` → `hubs.json`, `hubs.json.lock` (the root's list files, no hub directory).
    `git worktree list` → no worktree this round made; `git status --short` and `git stash list` empty.
    The round's scratch scripts `.scratch/killseed.sh` and `.scratch/killremove.sh` were removed by literal path.

## Rejected

- Probe 6: an `--allow-empty` task commit counts as landed, so `remove --remote --force` and `batten cancel --remove` delete it without an archive tag. That is the landed-work gate's design.
- Probe 9: clone A's `board sync` does not fetch the other clone's board when nothing is pending. That is board behaviour, outside the modules.

## Focus departures

- "Sandbox exception ... at a sibling worktree's sandbox hub path ... try a symlinked or `..`-relative path into the root and a hub moved out of the root."
  Not driven: no sibling worktree holds a sandbox root, and the round's time went to the leaks it found. The next round should drive them.
- "Joining ... Also join an origin that is mid-seed, one whose board branch is the legacy `lym-bolt`, and one whose default branch holds a warp binding."
  Only the seeded origin was joined.
- "Namespace leaks ... Settle which gitrepo and fabricengine call sites build a ref from a literal `refs/heads/` or `refs/tags/` on a records-store path, with an enumeration."
  The sweep was not made; F2 is one such site (`pushArchiveTag` builds its local ref and reads the code store's namespace). The next round should count the class.
- "Archive tags ... land a task" (a landed pair's tag) and "abandon a task": not driven, since landing needs a merge run the charter does not build from fabric verbs alone.
- "Two clones at once ... race `lym fabric push` from both ... with the first one's push held (`kill -STOP`) ... then from a second pair of hubs beside the first."
  Not driven; only the sequential board and prime-records pushes ran (probes 9, 16), so no concurrency claim is made.
- "Remote delete ... move the branch from clone B between A's read and A's delete": not driven.
- "A pair verb killed mid-operation": only `remove` during its archive tag push was killed; `add`, `push`, `sync`, `checkout`, `pull` and `reconcile` were not.
- "Namespace cache and layout read ... start the hub's reed server, rewire a store's fetch refspecs": the long-lived process case was not driven; only the missing-refspec case (probe 12).
- "Reconcile, prune and cleanup ... build tasks in each state (live, landed, abandoned, owned by another clone, origin-only) ... Sabotage the namespace in one call site."
  Only live and removed states were driven.
- Sabotage proofs (round context and rubric): not made. The review half changes no source file, so a namespace sabotage needs the fix half's build.
  Each finding instead rests on an observed result, and F5 carries a control run with the layout restored.
  F2 shows the namespace read in both stores in one drive: the records tag went to `refs/lym/tags/` and the code tag to `refs/tags/`.
  The rule that the review half changes no source file conflicts with the round context's sabotage request; the rule wins.
