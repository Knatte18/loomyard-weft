---
round: 3
focus:
  - "Re-drive the archive tag of a discarded task branch, with the code store's origin URL spelled three ways: equal to the records store's, as `file://<path>` of the same bare path, and a relative or symlinked spelling of it. Site: `archiveTagNamespaceAt`, `pushArchiveTag` and `archiveTaskTip` in `internal/fabricengine/archive.go`, `deleteArchivedTaskBranch` in `internal/fabricengine/taskbranchdiscard.go`. Claim under test: on a one-repository hub every archive tag of the pair lands under the records namespace root and never under plain `refs/tags/`, whatever the URL spelling. Repro: `lym harden sandbox new --single-repo`; `lym fabric add <slug>`, commit a real file, `lym fabric push`; `git -C <code store> remote set-url origin <spelling>`; `lym batten cancel <slug> --remove` from the prime, then `lym fabric cleanup --apply`; classify every ref of `git ls-remote <origin>`. Control: the same drive on a two-repository hub from `lym harden sandbox new`, where plain `refs/tags/archive/` is correct, and show its result differs. Failure: any `refs/tags/archive/` ref on origin, or a task branch deleted from origin with no archive tag it is recoverable from."
  - "Re-drive the split of archive-tag prefixes between the two stores of a one-repository hub. Site: `archiveTagNamespaceAt` in `internal/fabricengine/archive.go`, `remoteTagsUnder` in `internal/fabricengine/remoteleftover.go`, `readSideOwnerArchives` in `internal/fabricengine/ownerarchive.go`, `remoteTagRefs` in `internal/fabricengine/cleanupremotewarp.go`, `SlugRecordsTags` and `hasWeftEvidence`, and `reportengine`'s read of local `archive/` tags. Claim under test: the code store's archive tags and the records store's sit under distinct origin prefixes inside `refs/lym/`, each reader sees only its own side's tags, no store holds a commit of the other, and a fetch in the records store brings no code commit. Repro: discard a task branch with an unlanded code commit and remove its pair with `--remote`; then run `lym fabric clone <url> --into <dir>` for a second hub and list each of its stores' tags with `git for-each-ref refs/tags` and `git cat-file -t` on each tag's commit; then read `lym adopt --archives <root>` over owner-archive tags that the real `ArchiveOwnerTips` path pushed, staging a second machine's owner entry through the hubforge or board seam the sandbox allows, not tags pushed by hand; if the real path cannot be staged, name what blocked it. Failure: a tag listed twice or under the wrong side, a commit of one store in the other, or a code-side archive tag read as a records tag of the slug."
  - "Re-drive a killed teardown and the re-run its record names, on both halves. Site: `runRemoteTeardown` in `internal/fabricengine/remoteteardown.go`, `writePendingTeardown` and `pendingTeardownWayForward` in `internal/fabricengine/pendingteardown.go`, `archiveWeftTip` in `internal/fabricengine/archive.go`, `checkUnlandedRemoteTip` in `internal/fabricengine/destroy.go`. Claim under test: a teardown that dies at any phase, then is re-run through the command a reader prints, pushes every archive tag the first run made locally before any delete on origin, so no commit of an unlanded branch is left reachable only under a local tag. Repro, records half: a pair holding one unpushed records commit; an origin `pre-receive` hook that sleeps on `upd refs/lym/heads/<slug>-weft` and then declines; `lym fabric remove <slug> --remote`; `kill -9` the `lym fabric remote-teardown` child; `lym fabric status` and `lym fabric prune`; rerun the printed command; `git ls-remote` origin and `git cat-file -t` the local tip's sha12 in origin. Repro, discard half: a pushed pair with one unlanded code commit; the hook sleeps on `upd refs/lym/tags/archive/<slug>/` for the code tip's tag; `lym batten cancel <slug> --remove` from the prime; kill the child; rerun the printed command. Control: the same drive with no kill. Failure: an origin branch deleted while a commit of it sits on no origin ref, a stale lock, or a printed command that loops."
  - "Decide whether pushing every local `archive/<slug>/*` tag that origin lacks, at the slug's teardown, leaks tags an operator chose to keep local. Site: the push step of `runRemoteTeardown` and the keep-local path that `DropLocal` and `lym fabric cleanup` take in `internal/fabricengine`. Repro: make a pair, remove it without `--remote` so a local archive tag stays, then run `lym fabric remove <slug> --remote` and `lym fabric cleanup --apply --remote` on the same slug and on a slug whose archive tag came from another clone; list origin's tags before and after. Question: does origin receive a tag the operator kept local on purpose, and does any reader or `--help` text say so. Failure: a tag pushed with no verb output or documentation naming it."
  - "Re-drive a pending teardown record whose step failed without a kill. Site: `failedRemoteSteps` and the record clear in `internal/fabricengine/remoteteardown.go`, and the `status`, `prune`, `cleanup` and `reconcile` readers. Repro: an origin hook that declines the archive tag push; `lym fabric remove <slug> --remote`; read each reader and origin; type the printed commands exactly. Claim under test: a teardown with a failed step leaves a pending record that a reader names with the failed step's way forward, and clears it only after a run with no failed step. Failure: origin still holds a branch of the pair while no reader names it, or a printed command that fails when typed."
  - "Re-drive the records-store layout read for every verb that reads the board or the records upstream. Site: `Bolt.pullFirst` in `internal/fabricengine/bolt.go`, `pushClassified` in `internal/fabricengine/pushclassified.go`, `pullWeftArm` in `internal/fabricengine/pull.go`, `originNamespaceAt` in `internal/fabricengine/recordslayout.go`. Claim under test: with `remote.origin.fetch` unset in the records store, every verb that fetches, pulls, pushes or reads an upstream answers the records-store message with its restore commands, and none returns a raw `gitrepo:` error. Repro: `git -C <records store> config --unset-all remote.origin.fetch` in a one-repository sandbox hub; run `lym fabric reconcile` from a task worktree and from the prime, `lym fabric push`, `sync`, `pull`, `checkout`, `add`, `remove`, `cleanup --apply`, `prune --apply`, `lym board sync` and `lym adopt --archives <root>`; type the restore commands exactly and rerun each as the control. Also enumerate every call of `.Fetch()`, `Pull()`, `UpstreamSHA()`, `HasUnpushed()` and `HasUnpulled()` in `internal/fabricengine` with a one-line verdict on whether a layout read precedes it. Failure: a raw `gitrepo:` text, or a call with no layout read before it that a verb reaches."
  - "Drive a one-repository origin that already holds code-side archive tags in plain `refs/tags/archive/`, as an earlier binary leaves them. Claim under test: a reader verb names those refs, or the way forward to move them under the records namespace, and no teardown deletes a branch while the only tag of its commit sits in plain `refs/tags/`. Repro: make the tags by hand on a sandbox origin; run `lym fabric status`, `cleanup`, `cleanup --apply --remote`, `prune`, `reconcile`, `lym adopt --archives <root>` and a clone of that origin; classify what each prints and what a second clone's stores hold. Failure: a stray ref under plain `refs/tags/` that no verb names, or a clone that fetches it into the records store."
  - "Re-drive joining and the killed seed with the fixes since round 2. Site: the one-URL branch of `CloneHub` in `internal/fabricengine/clone.go`, `readOriginLayout` and `recordsprobe.go`, the `--into` handling of `lym fabric clone`. Repro: a bare code origin under the sandbox root with a logging `post-receive` hook that sleeps after `refs/lym/heads/main-weft`; `lym fabric clone --single-repo --shortname <s> <url> --into <dir>`; `kill -9` the clone; a plain `lym fabric clone <url> --into <other>` of the mid-seed origin; `lym fabric clone <url> --into <directory that does not exist>`; then each printed command typed exactly, `lym board sync`, the `--reset` clone and the join. Also kill the seed before its first push lands, and after the board lands with no upstream set. Failure: advice that describes the other layout, a probe error that names git when the directory is missing, a refusal that loops, or a printed command that fails when typed."
  - "Drive the regions no round has driven that a sandbox reaches, once the entries above are closed. Kill `lym fabric checkout`, `pull` and `reconcile` at the phases each has, by an origin hook that holds a push they make or a `kill -STOP` of the process between its fetch and its write; read a hub in the middle of its own clone from a second process; run a verb from a process that outlives a refspec rewiring if a reed or board server can be started under the sandbox root and then ended; drive the code-tip reset and hand-over paths (`internal/fabricengine/handover.go`, `internal/fabricengine/destroy.go`) as far as hubforge's topology seam allows against a live sandbox hub. List each region driven or the exact thing that blocked it. Failure: a ref under `refs/heads/` that belongs to the records, a lock that blocks a later verb, or a state no reader names."
  - "Re-drive each fixed key on a fresh one-repository hub as a claim to break, with a control and a sabotage proof for each: the archive prefix split, the pending record's tags, the layout read before the board pull, the seed's wording. A drive that stays green under a deliberately broken input (a respelled URL, a removed refspec, a killed child) has not entered the mechanism; name the instrumentation or control that shows each did. Run concurrent claims twice: a sequential control and a second hub beside the first, with the shortname `sbx` both record. Failure: a green drive with no proof it reached the code."
---

## Round context

Round 3 follows a round whose review found three BLOCKING findings and one MEDIUM finding, and whose fixes landed after the review.
The reviewer model must differ from round 2's, which was `fable`.
The entries above restate the claims for a fresh reviewer; none is a verdict, and none caps a severity.
The first entries re-drive what round 2 left standing; the last two cover the residue's shape and the regions no round has driven, in that order of priority.

## Modules and focus

The campaign hardens `./internal/fabricengine`, `./internal/fabriccli` and `./internal/gitrepo`, on the one-repository hub layout alone.
The operator's focus: the second hardener campaign on the layout that commit 57015a52 added.
One origin holds the code under `refs/heads/` and `refs/tags/`, and the records and board under `refs/lym/heads/` and `refs/lym/tags/`.
The focus binds: no round reviews outside it, and no focus file steers a round outside it.
The first campaign's closed findings are checked only to see that they hold, never re-opened unless a drive shows one undone.

## What to read

1. `internal/fabricengine/recordslayout.go` and `internal/gitrepo/refnamespace.go`, the only places the layout and namespace are spelled.
2. `internal/fabricengine/clone.go`, `pushCloneSeed` and its helpers in `internal/fabricengine/destroy.go`, and `refuseUnseededBoard` in `pushclassified.go`.
3. `internal/fabricengine/archive.go`, `remoteteardown.go`, `pendingteardown.go`, `taskbranchdiscard.go`, `ownerarchive.go`, `remoteleftover.go`,
   and `DeleteRemoteBranch` and `DeleteRemoteBranchLeased` in `internal/gitrepo/push.go`.
4. `internal/fabricengine/doc.go`, `internal/gitrepo/doc.go` and `internal/fabriccli/doc.go`.
5. `docs/overview.md`, `docs/getting-started.md`, the `PATTERN.md` entries that touch fabric, and `docs/research/single-repo/*/report.md`.
6. Each verb's `--help`: `lym fabric --help`, `lym harden sandbox --help`, `lym harden sandbox new --help`.
7. The existing integration tests as a map of what is already pinned; they run in the fix row's gates and are never the review.

## Layouts

Drive the one-repository hub that `lym harden sandbox new --single-repo` builds.
A two-repository hub from `lym harden sandbox new` is a control only, where a scenario needs the identity namespace to show a leak check sees a leak.
Make a second clone with `lym fabric clone <url> --into <dir>` from inside the sandbox root; every hub lives under the sandbox root.
Invoke only the `.dev-bin` lym the sandbox command prints, rebuilt after every source change.

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
  a board `sync` with nothing pending does not fetch;
  `fabric pull` on a code branch only ahead of an unmoved origin answering `warp remote diverged`, which both layouts share.
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

## Assignment

The review half only reviews and changes no source file.
Record each probe and its observed result in the review file as it is made.
Walk every entry as OK, WARN or FAIL, then improvise: verbs in orders nothing has tried, orphans of an operation that failed midway,
dead-but-present state, rapid churn of one slug from one clone and from two, and scope across two sandbox hubs of one shortname.
Prove each green drive entered its mechanism by a control or instrumentation, and never by editing source.
A concurrency claim needs a sequential control and a second hub's run of the same drive.
Give every finding its file and line, a concrete failure scenario, a severity, a class, a suggested fix and the repro that showed it.
