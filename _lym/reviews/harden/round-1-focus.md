---
round: 1
focus:
  - |
    Seed push order and rollback.
    Invariant: `lym fabric clone --single-repo --shortname <sn> <url>` pushes the prime's records branch first and the board `refs/lym/heads/main` last.
    A push that fails or is killed at either step leaves nothing on origin that reads as a hub, removes the hub directory, and restores the records branch to its observed state.
    Failure modes: a ref left under `refs/lym/heads/`, a hub directory left behind, a records branch not restored or restored over another writer's push, a raw git error with no way forward.
    Repro: make a bare origin under the sandbox root, once empty and once with a code commit on `main`, and once with a pre-existing `refs/lym/heads/main-weft`.
    Install a `pre-receive` hook in the bare that rejects a named ref, and run the clone once per ref (records branch, board); then make the hook sleep and kill the clone's `git push` mid-flight.
    After each run list origin with `git ls-remote`, list the clone's container directory, and run the same clone again to check it now succeeds.
    Also move origin's records branch between the precondition check and the push (a second clone pushes it), and check the lease loses honestly.
  - |
    Failure between hub creation and the seed push.
    Invariant: a `--single-repo` clone that fails after the hub exists but before the push (shortname, anchor, config materialization, a killed `lym` process) leaves no hub that a later reconcile or push could push unleased.
    Failure modes: a half-built hub with a records store and no origin ref, a later `lym fabric push` or `reconcile` that pushes the board without the lease or order.
    Repro: kill the clone process with SIGKILL at several points, such as between `git init` of the records store and the first push, and inspect the container and origin each time.
    Then run `lym fabric clone --single-repo` again in the same container, with and without `--reset`, and read what it says about the leftover.
  - |
    Joining.
    Invariant: `lym fabric clone <url>` on a seeded origin takes the one-repository layout with no flag.
    It writes the refspecs `+refs/lym/heads/*:refs/remotes/origin/*`, `+refs/lym/tags/*:refs/tags/*`, push `refs/heads/*:refs/lym/heads/*`, push `refs/tags/*:refs/lym/tags/*`, and `--no-tags`.
    It sets `branch.main.merge refs/lym/heads/main`, writes `lym.primeRecordsBranch`, and reads the board and the shortname from origin's board tip.
    Failure modes: a join that reads the URL as a weft (two-repository path), a missing key, a code tag fetched into the records store, a board that shows empty.
    Repro: seed a hub, add a board entry and a pair with a task branch, then join from a second container with `lym fabric clone <url>`.
    Compare `git config --get-regexp 'remote.origin|branch|lym'` in the records store with the seeding clone's, run `lym board list` and `lym fabric pairs` in the joined hub.
    Also join an origin whose default branch is not `main`, one with only the board and no records branch for the code default, and one holding a code tag.
  - |
    Refusals.
    Invariant: `--single-repo` on an origin that already holds a namespaced board, one whose default branch holds `.lym-warp` (a records repository), with a second URL, or an origin holding the old layout's `lym-bolt` branch or a root `.lym-anchor` or `_lym` refuses before it creates anything, and names its way forward.
    The `lym-bolt` refusal applies with and without `--single-repo`.
    Failure modes: a directory, a throwaway shallow clone, a stray temp directory or a hub left behind after the refusal; a refusal with no way forward; a refusal that does not fire when the flag order or `--reset` or `--into` changes.
    Repro: build each origin by hand in a bare under the sandbox root, run the clone from an empty container, and diff the container's contents and the origin's refs before and after.
    Also run each with `--reset` pointed at an existing hub and check the existing hub is untouched.
    Also run `--single-repo` with no `--shortname` and with an invalid one.
  - |
    Namespace leaks on origin.
    Invariant: no records ref ever lands under plain `refs/heads/` or `refs/tags/` on origin, and the code store never fetches, holds or pushes a records ref.
    Check every path: add, push, sync, pull, replay, hand-over, take-over, checkout, remove, teardown, archive, cleanup, prune, and the remote cleanup verbs.
    The archive tag of a discarded task branch was seen in plain `refs/tags/` in a sandbox trial; confirm or refute it, and say which store pushed it and under what name.
    Failure modes: `archive/<slug>/<sha>` or `<slug>-weft` under plain refs, a code-store ref in `refs/lym/`, a records tag in the code store's `refs/tags/`.
    Repro: on a fresh one-repository hub, drive a full pair lifecycle (`lym fabric add`, commits on both sides, `push`, `remove`, a discarded task branch, `cleanup`, `prune`), and run `git ls-remote origin` and `git for-each-ref` in both stores after every verb.
    Classify each ref as code or records by its origin and its prefix, and list any that sits on the wrong side.
    Also run the sweep on a task branch whose slug sorts with a `-weft` suffix, a branch name with slashes, and a branch named like a namespace segment such as `heads` or `tags`.
  - |
    Two clones of one hub.
    Invariant: two clones of the same origin, each pushing, pulling and writing to the board, converge on origin with no lost write and no ref under the wrong prefix.
    A lease that loses is reported as such, and the retry reads the winner's tip.
    Failure modes: a clobbered board write, a push that reports success while origin kept the other tip, a pull that reads `refs/heads/` instead of `refs/lym/heads/`, a dishonest `ok: true`.
    Repro: join a second hub from the same origin.
    Write a board entry and a record commit in each, push from both in both orders, then pull in both and compare tips with `git ls-remote`.
    Run the same drive from a second sandbox hub beside the first and keep a sequential control that does the writes one after the other.
    Also push the same branch from both at once with a delay hook in the bare, and push after one clone removed a pair the other still holds.
  - |
    Remote delete.
    Invariant: deleting a missing namespaced branch records no deletion and pushes nothing; deleting a present one removes `refs/lym/heads/<b>` on origin and the remote-tracking ref, and leaves `refs/heads/<b>` on origin alone.
    Failure modes: a recorded deletion of an absent ref, a delete aimed at plain `refs/heads/<b>` (the code branch of the same name), a stale remote-tracking ref, a leased delete that succeeds on a moved branch.
    Repro: create a pair, push it, then run `lym fabric remove` and `lym fabric cleanup` with the records branch present on origin, absent on origin, and present with a code branch of the same name.
    Check the mutation record each verb prints, and run each twice to check the second run is idempotent and records nothing.
    Also move the records branch on origin between the verb's read and its delete.
  - |
    Prime records branch.
    Invariant: `lym.primeRecordsBranch` is written once by the clone and no checkout moves it.
    A store without it fails closed with a way forward that names `git config`, and the orphan sweep keeps the prime's records branch.
    Failure modes: a sweep that deletes `main-weft`, a key rewritten by `checkout` or `add`, a cleanup that proceeds with the key unset, a key lost in a joined clone, a key that disagrees with a code default branch that was renamed.
    Repro: run `lym fabric cleanup` and `lym fabric prune` on a fresh hub, on a hub after `lym fabric checkout` of another branch, with the key unset (`git config --unset`), with the key empty, and with the key naming a branch that does not exist.
    Check `git for-each-ref refs/heads` in the records store and `git ls-remote` after each.
    Repeat on a joined clone.
  - |
    Layout detection.
    Invariant: the records store's fetch refspec decides the layout.
    An existing store whose remote config is unreadable (corrupt `config`, a `remote.origin.fetch` with a bad value, a missing origin remote) is an error, never read as two repositories.
    A path with no store reads as two repositories and is not cached for the process.
    Failure modes: a one-repository hub silently driven with identity refs after a config fault, a verb that pushes `refs/heads/<b>-weft` to origin, a cached answer that goes stale when a store appears or changes within one process.
    Repro: on a one-repository hub corrupt the records store's config in turn (truncate it, delete `remote.origin.fetch`, point it at `+refs/heads/*`, remove the `origin` remote), then run `lym fabric status`, `push`, `pull`, `add`, `remove` and `cleanup`, and check origin after each.
    Also swap the refspec to the two-repository form and back between two verbs, and run a verb that spans both stores while the code store has no `.git`.
  - |
    Config read and upstream.
    Invariant: git's `@{u}` works on a namespaced records branch (`branch.<b>.merge refs/lym/heads/<b>`), and `gitrepo`'s tolerant config read accepts that value, for branches the clone made, branches `add` made, and branches a pull or adopt created.
    Failure modes: a go-git path that rejects `refs/lym/heads/<b>`, an upstream that reads as unset, an ahead/behind count that is wrong, a push that falls back to an unleased default.
    Repro: on a hub with several pairs run `git rev-parse @{u}` and `git status -sb` in the records store on each branch, then `lym fabric status`, `pairs`, `pull` and `push`.
    Set `branch.<b>.merge` to a value outside both namespaces and to an empty value, and read what each verb says.
    Also give a records branch with an odd name (dots, slashes, a long slug) and check that the config read and the push destination agree.
  - |
    Crash and rebirth of a one-repository pair.
    Invariant: a pair verb killed mid-operation leaves no orphan ref on origin in the wrong namespace and no half-registered pair, and the next verb recovers or says exactly what to do.
    Failure modes: a records branch pushed with no code branch, an archive tag pushed twice, a `remove` that deleted the code branch but left the records branch and the board entry, a reconcile that reads the namespace wrongly.
    Repro: kill `lym fabric add`, `push`, `remove` and `cleanup` with SIGKILL after each git step (use a `PATH` shim for `git` that kills the process on a chosen subcommand), then run `lym fabric status`, `reconcile`, `prune` and the same verb again.
    Also a hand-over and a take-over interrupted the same way, and a push whose origin rejects the namespaced ref.
  - |
    Environment and origin variations.
    Invariant: the layout holds when origin is a local bare path, a `file://` URL, a URL with a trailing slash or `.git`, and an origin whose HEAD names a non-`main` default branch.
    Failure modes: a hub name derived wrongly, a ref listing that misses `refs/lym/*`, a board branch read from the code default.
    Repro: seed and join on each URL form and, in a bare whose HEAD is `trunk`, with `main` also present, and check `lym fabric clone`, `status` and `push` on each.
---

## Modules and focus

The modules are `internal/fabricengine`, `internal/fabriccli` and `internal/gitrepo`.
The operator's focus binds this campaign: only the one-repository hub layout that commit 57015a52 added.
Origin holds the code under `refs/heads/` and `refs/tags/`, and the records and the board under `refs/lym/heads/` and `refs/lym/tags/`.
Everything else in fabric, the two-repository layout included, is out of scope, and no round steers outside this focus.

Code paths in scope:

- `internal/fabricengine/recordslayout.go`: the namespace constants, `originNamespaceAt`, `OriginRefNamesAt`, `readOriginLayout` and `refuseOneURLClone`.
- `internal/fabricengine/clone.go`: `CloneHub`'s one-URL branch, `initRecordsStore`, `readJoinedBoardRecords`, `PushCloneBoard`, `AbandonSeededClone`.
- `internal/fabricengine/destroy.go`: `pushCloneSeed`, `restoreSeededRecordsBranch`, `checkSeedPush`, `seedPushCause`.
- Every site that hands the namespace to a push, a listing, a fetch or a delete:
  `archive.go`, `taskbranchdiscard.go`, `ownerarchive.go`, `remoteteardown.go`, `cleanup.go`, `cleanupremotewarp.go`,
  `originreconcile.go`, `remoteleftover.go`, `handover.go`, `takeoverwrite.go`, `add.go`, `checkout.go`, `pushclassified.go`, `resetsubpathcheck.go`, `weftbranch.go`.
  The grep `grep -rn "originNamespaceAt\|RefNamespace\|refs/lym\|ns\.Branch\|ns\.Tag" internal --include=*.go` enumerates them.
- `internal/fabriccli/clone.go` and `fabric.go`: the `--single-repo` flag, its usage text and its refusals.
- `internal/gitrepo`: `RefNamespace` (`refnamespace.go`), `push.go`, `pushdestination.go`, `pushretry.go`, `remote.go` (`RemoteFetchRefspecs`), `refreads.go`, `blobread.go` (`UpstreamSHA`), the tolerant config read, and `merge.go`.

## What to read

Read first:

- `pattern/PATTERN-single-repo-hub.md`, the `PATTERN-code-conventions.md` entry on lym-owned records-namespace branches, and `README.md` and `docs/getting-started.md` on the layout and its hazards.
- `internal/fabricengine/doc.go` and `internal/fabriccli/doc.go` and `internal/gitrepo/doc.go`.
- `docs/research/single-repo/sandbox-1-setup-2026-11-21/report.md`, `sandbox-2-tasks-2026-11-21/report.md` and `sandbox-3-reclone-2026-11-21/report.md`: trials against a real GitHub remote.
  They are context for the seen archive-tag symptom; the round settles it on its own sandbox hub.
- The code paths listed above, and `internal/hubforge/hub.go` for the one-repository builder.
- Verbs: `lym fabric clone --help`, and `--help` of `add`, `push`, `pull`, `remove`, `cleanup`, `prune`, `reconcile`, `checkout`, `status`, `pairs`; `lym harden sandbox --help` for `new`, `list` and `clear`.
- Existing tests, for what they leave undriven: `internal/fabricengine/clone_singlerepo_integration_test.go`, `clone_onerepository_test.go`, `recordslayout_integration_test.go`, `onerepository_lifecycle_integration_test.go`, `remoteteardown_namespace_integration_test.go`;
  `internal/gitrepo/deleteremotebranch_integration_test.go`, `push_destination_integration_test.go`, `remote_integration_test.go`.
  Every live test of these modules is tier 1 or below; none of them is the review.

## Layouts

Drive the one-repository hub, built by `lym harden sandbox new --single-repo`.
That is the layout under review, and the sandbox hub is cloned from one local bare repository.

The two-repository hub that `lym harden sandbox new` builds is out of scope and no round drives it as a subject.
A round may build one only as a control to see that a behaviour belongs to the one-repository layout, and records it as a control.

Scenarios that need an origin the sandbox hub does not have (an empty origin, a code-only origin, an origin with a namespaced board, an old-layout origin) use bare repositories and containers made under the sandbox root, cloned with the `.dev-bin` lym the sandbox prints.
Some `lym fabric` verbs refuse a task-session role; when a verb is refused for that reason, record the refusal text and report the blocked scenario as "could not verify" with the reason.

## Out of scope

- The two-repository layout and every behaviour that holds only in it.
  Its code paths in the modules are correct to keep a code-repository archive tag in plain `refs/tags/`; the question here is only whether that holds for a shared origin.
- The rest of fabric: junction wiring, anchors, merge conflict handling, the dobby, the board's verbs, reed and every module outside the three named.
  A fabric verb is driven only to read its effect on origin's refs, its records store and its namespace.
- Conversion of the earlier one-repository layout (`lym-bolt`): there is none, and a refusal that names the cleanup is the correct result.
- A host that refuses refs outside `refs/heads/` and `refs/tags/`: the seed's refusal that names a two-repository hub is correct.
  The sandbox's local bare origins accept any ref.
- GitHub-owned refs such as `refs/pull/*`, a branch list that GitHub renders, and a plan-gated rules API: the sandbox has no GitHub remote.
- Absences that are correct: a code store holds no `refs/remotes/origin/*-weft` and no `refs/lym/` ref; origin's `refs/heads/` holds code branches only; a joined hub writes no `lym.primeRecordsBranch` into the code store.
- A timeout under the concurrent copies' CPU peg: a diagnostic, never a defect.
- The existing tests: rerunning them checks a fix and is never a campaign item.

## Live-substrate cost declaration

No live test or verb of these modules spawns a real LLM session.
Every drive is a plain `git` and `lym` process on local bare repositories, so the campaign names no tier-3 test and bans none.
A fabric verb that starts a reed server or a tmux pane for the sandbox hub belongs to the teardown check below.

## Teardown check

Run these at the end of the round and record the output.
The sandbox root is `<worktree>/.lym/harden-sandbox`.

```
lym harden sandbox clear
pgrep -af "harden-sandbox" || echo "no process names the sandbox root"
tmux ls 2>&1 | grep -i "harden-sandbox" || echo "no sandbox tmux session"
ls .lym/harden-sandbox 2>&1
git -C . for-each-ref refs/lym refs/tags | head
lym harden sandbox list
```

Zero stray state means: `clear` ends nothing it did not already list, no process or tmux session names the root, the root holds no hub, bare origin or container directory, the worktree's own repository holds no ref the round pushed, and `lym harden sandbox list` lists no hub.
Use the `.dev-bin` lym for hub verbs, and the lym on the path for `harden sandbox` verbs.

## Round context

Round 1 is a full adversarial review of the scope above, on a one-repository sandbox hub.

- Walk every focus entry and record OK, WARN or FAIL for each, with the repro steps that showed it.
- Settle the archive-tag question first: on a fresh `--single-repo` hub, discard a task branch (create a pair, push, remove it so its task branch is archived), and list where the archive tag of the task branch and the archive tag of the records branch landed on origin.
  A finding states the store that pushed each tag and the ref it landed under.
- Then improvise beyond the entries: scenarios the entries do not name, verbs combined in orders nothing has tried, a second hub running the same drive beside the first, and a sabotage proof for each green drive of a mechanism (the namespace parameter, the lease, the order of the seed, the existence check before a delete).
- Prove each concurrency claim with a sequential control and a second-hub run.
- A finding gives its file and line, a concrete failure scenario with the repro steps, a severity of BLOCKING, MEDIUM, LOW or NIT, a suggested fix, and CONFIRMED or PLAUSIBLE.
  Record every finding, whatever its size.
