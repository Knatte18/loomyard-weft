---
round: 4
focus:
  - "Drive the discard half of a killed teardown, and the re-run its record names. Site: `runRemoteTeardown` in `internal/fabricengine/remoteteardown.go`, `writePendingTeardown` and `pendingTeardownWayForward` in `internal/fabricengine/pendingteardown.go`, `archiveTaskTip` in `internal/fabricengine/archive.go`, `deleteArchivedTaskBranch` in `internal/fabricengine/taskbranchdiscard.go`. Claim under test: a discard that dies after the local archive tag is made and before it reaches origin, then re-run through the printed command, pushes the tag before it deletes `refs/heads/<slug>` on origin, so no code commit is left reachable only under a local tag. Repro: `lym harden sandbox new --single-repo`; `lym fabric add <slug>`, commit a real file, `lym fabric push`; an origin `pre-receive` hook that sleeps on `refs/lym/code-tags/archive/<slug>/` and then declines; `lym batten cancel <slug> --remove` from the prime; `kill -9` the `lym fabric remote-teardown` child; read `lym fabric status` and `lym fabric prune`; type the printed command exactly; `git ls-remote` origin and `git cat-file -t` the code tip's sha12 in origin. Control: the same drive with no kill. Failure: `refs/heads/<slug>` deleted on origin while the tip sits on no origin ref, a stale lock, a reader that names no way forward, or a printed command that fails or loops."
  - "Decide whether pushing every local `archive/<slug>/*` tag that origin lacks, at a slug's `--remote` teardown, leaks a tag an operator chose to keep local. Site: the push step of `runRemoteTeardown` and the keep-local path of `DropLocal` and `lym fabric cleanup` in `internal/fabricengine`. Repro: make a pair, remove it without `--remote` so a local archive tag stays; then run `lym fabric remove <slug> --remote`, and `lym fabric cleanup --apply --remote`, on that slug and on a slug whose archive tag came from another clone; list origin's tags before and after each. Question: does origin receive a tag the operator kept local on purpose, and does any verb output or `--help` text say so. Failure: a tag pushed with no verb output or documentation naming it."
  - "Drive a pending teardown record whose step failed without a kill. Site: `failedRemoteSteps` and the record clear in `internal/fabricengine/remoteteardown.go`, and the `status`, `prune`, `cleanup` and `reconcile` readers. Repro: an origin hook that declines the archive tag push; `lym fabric remove <slug> --remote`; read each reader and `git ls-remote` origin; type each printed command exactly; then remove the hook and re-run. Claim under test: a teardown with a failed step leaves a pending record that a reader names with the failed step's way forward, and clears it only after a run with no failed step. Failure: origin still holds a branch of the pair while no reader names it, or a printed command that fails when typed."
  - "Drive `lym adopt --archives <entry id>` over owner-archive tags that the real `ArchiveOwnerTips` path pushed. Site: `readSideOwnerArchives` in `internal/fabricengine/ownerarchive.go`, `remoteTagsUnder` in `internal/fabricengine/remoteleftover.go`, `remoteTagRefs` in `internal/fabricengine/cleanupremotewarp.go`. Claim under test: the owner-archive tags of a one-repository hub sit under one origin prefix, each store's reader sees only its own side's tags, and a second clone's stores hold no commit of the other side. Repro: stage a second machine's owner entry through the board or hubforge seam the sandbox allows and let the real path push its tags, never tags made by hand; `lym fabric clone <url> --into <dir>` for a second hub; list each store's tags with `git for-each-ref refs/tags` and `git cat-file -t` on each tag's commit; run `lym adopt --archives <entry id>` in both hubs. If the real path cannot be staged, name what blocked it. Failure: a tag listed twice or under the wrong side, a commit of one store in the other, or a code-side archive tag read as a records tag of the slug."
  - "Decide what a one-repository origin holding plain `refs/tags/archive/` tags, as an earlier binary leaves them, shows a reader. Claim under test: a reader verb names those refs or the way forward to move them under the records namespace, and no teardown deletes a branch while the only tag of its commit sits in plain `refs/tags/`. Repro: make a tag by hand on a sandbox origin that points at an unlanded code commit; run `lym fabric status`, `cleanup`, `cleanup --apply --remote`, `prune`, `reconcile`, and a `lym fabric clone` of that origin; classify what each prints and what the second clone's stores hold. Failure: a stray ref under plain `refs/tags/` that no verb names, a clone that fetches it into the records store, or a teardown that deletes a branch whose only tag is plain."
  - "Re-drive the archive tag of a discarded branch with the code store's origin URL spelled relatively, and again on a two-repository hub as a control. Site: `archiveTagNamespaceAt` and `pushArchiveTag` in `internal/fabricengine/archive.go`. Claim under test: on a one-repository hub every archive tag of the pair lands under the records namespace root for a relative spelling of origin's path, and on a two-repository hub from `lym harden sandbox new` the plain `refs/tags/archive/` placement is correct and differs from the one-repository result. Repro: `git -C <code store> remote set-url origin <relative path>`; `lym fabric add <slug>`, commit, `lym fabric push`; `lym batten cancel <slug> --remove` from the prime; classify every ref of `git ls-remote <origin>`. Failure: any `refs/tags/archive/` ref on origin of the one-repository hub, or a branch deleted from origin with no archive tag it is recoverable from."
  - "Enumerate every call of `.Fetch()`, `Pull()`, `UpstreamSHA()`, `HasUnpushed()` and `HasUnpulled()` in `internal/fabricengine` and `internal/fabriccli`, with a one-line verdict on whether a records-store layout read precedes it. Report the total, the command that enumerated it and what that command cannot see. Then drive each call a verb reaches with `remote.origin.fetch` unset in the records store (`git -C <records store> config --unset-all remote.origin.fetch`). Failure: a raw `gitrepo:` text, or a verb-reachable call with no layout read before it."
  - "Drive the regions no round has driven that a sandbox reaches: kill `lym fabric checkout`, `pull` and `reconcile` at each phase they have, by an origin hook that holds a push they make or a `kill -STOP` between the fetch and the write; read a hub in the middle of its own clone from a second process; drive the code-tip reset and hand-over paths in `internal/fabricengine/handover.go` and `internal/fabricengine/destroy.go` as far as hubforge's topology seam allows. List each region driven or the exact thing that blocked it. Failure: a ref under `refs/heads/` that belongs to the records, a lock that blocks a later verb, or a state no reader names."
  - "For each of the archive prefix split, the pending record's tags, the layout read before the board pull and the seed's wording, give a sabotage proof and a control: a drive that stays green under a deliberately broken input (a respelled URL, a removed refspec, a killed child) has not entered the mechanism, so name the instrumentation or control that shows it did, never by editing source. Run each concurrency claim twice: a sequential control and a second hub beside the first, with the shortname `sbx` both record. Failure: a green drive with no proof it reached the code."
---

## Round context

Round 4 follows a round whose review found nothing and left the entries above undriven or driven in part.
The reviewer model must differ from round 3's, which was `sonnet`.
Round 3's live report was red: `harden-live` passed and started no test binary.
The round's live gate must start the campaign's live tests; a run that starts none is not green.
The entries restate claims for a fresh reviewer; none is a verdict, and none caps a severity.
The operator's ruling is that a round never approves with a focus entry undriven: walk every entry as OK, WARN or FAIL, driven live.

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
  `fabric pull` on a code branch only ahead of an unmoved origin answering `warp remote diverged`, which both layouts share;
  `fabric remove <slug> --remote` leaves `refs/heads/<slug>` on origin, since no remove verb deletes the code branch.
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
