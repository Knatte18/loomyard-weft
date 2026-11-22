---
round: 2
focus:
  - "Re-drive the archive tag of a discarded task branch. Site: `pushArchiveTag` and `archiveTaskTip` in `internal/fabricengine/archive.go`, `deleteArchivedTaskBranch` and its origin-side archive check in `internal/fabricengine/taskbranchdiscard.go`. Claim under test: on a one-repository hub the tag lands under `refs/lym/tags/archive/<slug>/<sha12>` and never under plain `refs/tags/`, and the origin delete of the task branch reads the same namespaced tag. Repro: `lym harden sandbox new --single-repo`; `lym fabric add <slug>`, commit a real file, push the code branch, then `lym batten cancel <slug> --remove` from the prime and `lym fabric cleanup --apply`; list origin with `git ls-remote <origin>` and classify every ref; then clone a second hub with `lym fabric clone <url> --into <dir>` and list its code store's tags. Failure: any `refs/tags/archive/` ref on origin or in the second hub's code store, or a task branch deleted from origin with no archive tag it can be recovered from. Prove the drive reached the mechanism: run the same drive on a two-repository hub as a control where the identity namespace is correct, or instrument the push, and show the control differs."
  - "Count the class of call sites that build a ref from a literal `refs/heads/` or `refs/tags/`, or read a store's namespace, on a path that can reach a records store or a code store of a one-repository hub. Scope: `internal/fabricengine` and `internal/gitrepo`, with `originNamespaceAt`, `Namespace.Tag` and `Namespace.Branch` as the stable identifiers. List every site in a table: the correct ones with the reason, the wrong ones as findings, then the total, the reproducible command that enumerated it (name the grep patterns) and what that command cannot see (dynamic ref strings, refs built in tests, callers through interfaces). The round states no count in advance. Failure: a site that pushes, fetches or deletes a records ref under identity or a code ref under the records namespace."
  - "Re-drive a records store whose fetch refspecs are removed. Site: the push job in `internal/fabricengine/pushclassified.go`, `pullWeftArm` in `internal/fabricengine/pull.go`, `originNamespaceAt` and `recordsStoreClaim`. Claim under test: every pushing and pulling verb, including the detached push that `lym fabric sync` and `lym fabric add` spawn, refuses with the records-store message and its restore commands, records the refusal where `lym fabric status` lists it, and never reports success or moves nothing silently. Repro: in a task worktree's hub from `lym harden sandbox new --single-repo`, run `git -C <records store> config --unset-all remote.origin.fetch`; run `lym fabric push`, `lym fabric sync`, `lym fabric pull`, `lym fabric checkout`, `lym fabric reconcile` and `lym board sync`; advance origin's `refs/lym/heads/<branch>` by hand before the pull; read each verb's JSON, the hub log and `lym fabric status`; then type the restore commands exactly and rerun each verb as the control. Failure: a raw `gitrepo:` error with no way forward, `ok: true` with no mutation, an empty `failed_pushes` after a failed detached push, or restore commands that do not work when typed."
  - "Re-drive a rejected first push. Site: `unpushedCommits` and `failRejection` in `internal/fabricengine/pushclassified.go`. Claim under test: when a records branch has no upstream and its first push is rejected, the way forward names the branch's own local commits and never says there are none before advising a command that drops commits. Repro: two clones of one origin (`lym fabric clone <url> --into <dir>` inside the sandbox root); `lym fabric add <slug>` with the same slug in both; `lym fabric push` in the second; compare the refusal's listed commits with `git rev-list --count origin/<slug>-weft..HEAD` and `git config --get-regexp 'branch\\.<slug>'`. Failure: a count of local commits in the refusal that differs from git's, or advice that drops a commit it did not list."
  - "Re-drive a teardown child that dies. Site: `RunTeardownChild` in `internal/fabricengine/remoteteardown.go`, `warnWhenEnded` in `internal/fabriccli/teardownchild.go`, the teardown lock beside `_bolt/.lym/teardown/`, and the `status`, `prune`, `cleanup` and `reconcile` readers. Claim under test: after `kill -9` of the `remote-teardown` child of `lym fabric remove <slug> --remote`, at each phase (before the archive tag push, during it with a sleeping `pre-receive` hook in origin on `refs/lym/tags/archive/*`, between the tag push and the branch deletes, after the code-branch delete and before the records-branch delete), a reader verb names the slug and the command that clears it, and rerunning that command clears origin without a loop. Repro: build the pair with `lym fabric add` and `lym fabric push`, run the remove in the background, kill the child at the phase, read origin with `git ls-remote`, run each reader verb, rerun the printed command, read origin again. Failure: an origin branch of the pair that survives with no reader naming it, a stale lock that blocks the rerun, or a way forward that loops. Also kill `lym fabric add`, `push`, `sync`, `checkout`, `pull` and `reconcile` at the phases the verb has (before the records push, between the records push and the code push, after both) and apply the same reader check; round 1 drove no verb but the remove."
  - "Re-drive the sandbox guard for pushes. Site: `sandboxedRoleRefusal` in `internal/fabriccli/roleguard.go` and the refusal rows in `internal/fabriccli/cli_refusals_test.go`. Claim under test: a task session's mutating `lym fabric` verb runs only against a hub under its own worktree's `.lym/harden-sandbox`, whatever path flags it passes, and the refusal text names `lym harden sandbox new` as a way forward. Repro: from this worktree's real hub, run `lym fabric push --weft-path <path> --warp-path <path>` with the paths of (a) a sandbox hub's own pair, (b) a pair outside the sandbox root, (c) a path reached through a symlink into the root, (d) a `..`-relative path that leaves the root, (e) a pair of a sibling worktree's sandbox root if one exists, and (f) a hub moved out of the root after it was made; run each other mutating verb (add, pull, sync, checkout, remove, cleanup --apply, prune --apply, reconcile, clone, hand-over, replay) from the real hub and from a sandbox hub. Use a disposable sandbox hub as the target of (b) and (f), never the real hub's own stores. Failure: any ref or file changed outside the sandbox root, or a refusal with no way forward. Prove the guard is reached by running a verb that must pass from inside the hub as the control."
  - "Re-drive joining and the killed seed on origins in states round 1 did not join. Site: `internal/fabricengine/clone.go`, `pushCloneSeed` and its helpers in `internal/fabricengine/destroy.go`, `refuseUnseededBoard` in `internal/fabricengine/pushclassified.go`. Join with `lym fabric clone <url> --into <dir>` an origin that is mid-seed (records branch pushed, board not), one whose board branch is the legacy `lym-bolt`, one whose default branch holds a warp binding, one holding a board of another clone with no common history, and one whose records branch sits at another tip; compare each join's records store with `git config --get-regexp` against a seeded hub's. Re-drive a `kill -9` of `lym fabric clone --single-repo` at each point of the seed (a sleeping `post-receive` hook on `refs/lym/heads/main` and on `refs/lym/heads/main-weft`), then `lym board sync`, each printed command typed exactly, the `--reset` clone and the join clone. Failure: a refusal that loops back to one it gave, a false statement about what origin holds, a hub that joined with the identity layout, or advice whose commands fail when typed."
  - "Drive two clones of one origin at once, with a sequential control and a second pair of hubs. Site: `internal/gitrepo/push.go`, `pushretry.go`, `remoterefslock.go`, the board's sync and the prime records push. Repro: clones A and B; hold A's `lym fabric push` with `kill -STOP` during a pre-receive hook that sleeps, push from B, release A; the same drive run sequentially as the control; the same drive again in a second pair of hubs beside the first, with a different shortname. Also add a task of the same slug in both, write a board entry in each and sync both, push the prime's records branch from both, and move a remote branch from B between A's read and A's delete in a teardown. Failure: a lost lease reported as success or as a bad token, a board write lost, a push that overwrites the other clone's records branch, a deletion recorded for a ref that never existed, a code branch of the same short name deleted, or a refusal with no way forward. A claim without the sequential control and the second pair of hubs is not graded."
  - "Drive reconcile, prune and cleanup against origin in each task state: live, landed, abandoned, owned by another clone, origin-only, and with a records branch and a code branch of one short name on origin. Run each verb judge-only and then moving; read origin and the hub after each. Site: `internal/fabricengine/reconcile.go`, `originreconcile*.go`, `prune.go`, `cleanup.go`, `cleanupremotewarp.go`. Failure: a records branch judged against a code branch of the same name, a remote-only run torn down while its code branch is live, an ended entry torn down although another machine owns it, an archive tag deleted, or an orphan (records branch with no code branch, code branch with no records branch, archive tag with no run) that no verb names. Prove the drive enters the namespace read: show a control where the same-name code branch is absent and the verb's verdict differs."
  - "Drive the long-lived namespace cache. Site: `originNamespaceAt` and its process cache in `internal/fabricengine`. Claim under test: a process that outlives a clone or a refspec rewiring never keeps a stale identity answer, and a verb run while a second hub's clone is in flight never errors on a hub in a transient state. Repro: start the hub's reed server, rewire a store's fetch refspecs, delete the prime key by hand, run a push; run a verb while a second sandbox hub is being cloned; run every pushing verb with `remote.origin.fetch` removed. Failure: a records store read as identity after its refspecs were wired, or a records push into `refs/heads/`."
---

## Round context

Round 2 is the residual round of the second fabric hardener campaign.
Round 1 found seven findings, and six fix commits landed after its review: an archive tag namespace fix, a records-layout read before pushes and pulls,
the unpushed-commit listing with no upstream, a pending-teardown record for a killed teardown child, a push guard that checks injected paths, and the sandbox refusal wording.
No fresh reviewer has seen those fixes, so each entry above re-drives one of them as a claim to be broken, never as a thing to take as closed.
The reviewer model must differ from round 1's, which was `opus`.

## Modules and focus

The campaign hardens `./internal/fabricengine`, `./internal/fabriccli` and `./internal/gitrepo`, on the one-repository hub layout alone.
The operator's focus: the second hardener campaign on the layout that commit 57015a52 added.
One origin holds the code under `refs/heads/` and `refs/tags/`, and the records and board under `refs/lym/heads/` and `refs/lym/tags/`.
The focus binds: no round reviews outside it, and no focus file steers a round outside it.
The first campaign's closed findings are checked only to see that they hold, never re-opened unless a drive shows one undone.

## What to read

1. `internal/fabricengine/recordslayout.go` and `internal/gitrepo/refnamespace.go`, the only places the layout and namespace are spelled.
2. `internal/fabricengine/clone.go`, `pushCloneSeed` and its helpers in `internal/fabricengine/destroy.go`, and `refuseUnseededBoard` in `pushclassified.go`.
3. `internal/fabricengine/archive.go`, `remoteteardown.go`, `taskbranchdiscard.go`, and `DeleteRemoteBranch` and `DeleteRemoteBranchLeased` in `internal/gitrepo/push.go`.
4. `internal/fabricengine/doc.go`, `internal/gitrepo/doc.go` and `internal/fabriccli/doc.go`.
5. `docs/overview.md`, `docs/getting-started.md`, the `PATTERN.md` entries that touch fabric, and `docs/research/single-repo/*/report.md`.
6. Each verb's `--help`: `lym fabric --help`, `lym harden sandbox --help`, `lym harden sandbox new --help`.
7. The existing integration tests as a map of what is already pinned; they run in the fix row's gates and are never the review.

## Layouts

Drive the one-repository hub that `lym harden sandbox new --single-repo` builds.
A two-repository hub from `lym harden sandbox new` is a control only, where a scenario needs the identity namespace to show a leak check sees a leak.
Make a second clone with `lym fabric clone <url> --into <dir>` from inside the sandbox root; every hub lives under the sandbox root.
Invoke only the `.dev-bin` lym the sandbox command prints.

## Out of scope

- The two-repository layout and everything in fabric that is not on a one-repository path.
- Every module outside the three above, board, batten, webster, reed and loom internals included, except where a fabric verb's result is read through them.
- A real GitHub remote; name host-side behaviour under Coverage, never as a finding.
- Correct absences, never flagged: a branch with no origin tracking ref is not judged by `ReconcileOrigin`;
  a judge-only reconcile writes no `FETCH_HEAD`; `main-weft` does not advance on landing;
  a one-URL `--single-repo` clone is refused when origin already holds a board, with `lym fabric clone <url>` as its way forward;
  the code store of a one-repository hub holds no ref from the records namespace;
  a mutating `lym fabric` verb run outside the sandbox root from a task session is refused by design;
  an `--allow-empty` task commit counts as landed, so deleting it needs no archive tag;
  a board `sync` with nothing pending does not fetch.
- A timeout under the concurrent copies' CPU peg is a diagnostic, never a defect.
- `lym selfreport create` is never run.

## Live-substrate cost declaration

No live verb or test of these modules spawns a real LLM session; there is no tier-3 execution ban.
Build task states with `lym fabric add`, `lym board` verbs and plain commits, never a `lym beam` or `lym batten` run that starts a real session.
A round that finds it needs a real session names why first and runs one at a time.

## Teardown check

Run at the end of the round and record every output:

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

No hub may stay listed, no process may name the sandbox root, and no directory may stay under `.lym/harden-sandbox`.
No worktree, stash or modified file other than the round's review file and the fix row's commits may remain.
Remove each scratch file by its literal path under `.scratch/`.

## Priority when the round cannot cover every entry

Cover the entries in this order, and name what was left:
the literal-ref call-site count, two clones at once, a pair verb killed mid-operation, a remote delete with the branch moved from the second clone,
reconcile, prune and cleanup over each task state, the sandbox guard at a sibling root and through symlinked, `..`-relative and moved paths,
then joining a mid-seed origin and the killed seed.
Re-drive the archive tag and push-guard fixes live on a fresh one-repository sandbox hub first.

## Assignment

The review half only reviews and changes no source file.
Record each probe and its observed result in the review file as it is made.
Walk every entry as OK, WARN or FAIL, then improvise: verbs in orders nothing has tried, orphans of an operation that failed midway,
dead-but-present state, rapid churn of one slug from one clone and from two, and scope across two sandbox hubs of one shortname.
Prove each green drive entered its mechanism by a control or instrumentation, and never by editing source.
A concurrency claim needs a sequential control and a second hub's run of the same drive.
Give every finding its file and line, a concrete failure scenario, a severity, a class, a suggested fix and the repro that showed it.
