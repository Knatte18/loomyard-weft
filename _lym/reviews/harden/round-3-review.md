---
verdict: APPROVED
prior:
  - id: F6
    status: resolved
    answer: "Driven on a one-repository sandbox hub: a post-receive that kills the client after refs/lym/heads/main-weft applied leaves origin with no refs/lym/ ref (reference-transaction log shows the create and the delete), where the build before 704c7ae leaves the ref behind. A pre-receive that kills the client and exits non-zero keeps the hub and names both refs and the ls-remote and push --delete commands."
  - id: F7
    status: resolved
    answer: "With the prime pair on alt, every refusal in cleanup --remote names a git config way forward with the suffix rendered through RecordsBranchName as <clone-time-code-branch>-weft; unset, a missing branch, the board branch main and a nested name are refused, a padded right value is trimmed and protects main-weft. No production file outside internal/weftname spells the suffix as a string literal."
  - id: F8
    status: resolved
    answer: "A records store whose only mark is a branch.<b>.merge value under refs/lym/ is refused by lym board sync and lym fabric cleanup --remote, with origin untouched, including with origin's code default branch named trunk. The refusal names the setting and the fetch-refspec way forward."
---

No finding is raised.
Every drive below used the `.dev-bin` lym built from HEAD `8a8ff4024`, and every seed ran as `.dev-bin/lym harden sandbox new --single-repo`.
The comparison build of `704c7aea6~1` lives in the scratchpad as `lym-old`.

## Probes

Hooks were planted through `GIT_TEMPLATE_DIR`, which the sandbox's bare copy honours.
Each template also holds a `reference-transaction` hook that logs every committed `refs/lym/` change on origin, so the end state is the net effect of its log.
`sandbox new` removes the whole container when the seed fails, so origin cannot be listed after the call returns;
the log is the record of origin's end state, and the error text is the record of whether the hub was kept.

1. Seed push killed after origin applied it (`post-receive`, `kill -9` of the client `git push`):
   - Board ref: the log shows the board created, then deleted, then the records branch deleted.
     The error is "the seed push of the board failed … way forward: run the same clone again", with no kept-hub text. OK.
   - Records ref: the log shows the records branch created, then deleted. The error is the same rerun way forward. OK.
   - The same records drive on `lym-old`: the log shows the create and no delete, so origin keeps `refs/lym/heads/main-weft` with no hub.
     The drive reaches the mechanism F6 fixed.
2. Seed push killed and declined (`pre-receive`, kill then exit 1):
   - Board ref: origin keeps the records branch only. The error says the push "ended without origin's answer", the hub is kept, origin may hold `refs/lym/heads/main` and `refs/lym/heads/main-weft`, and it names `ls-remote` and `push origin --delete <ref>`. OK.
   - Records ref: origin holds nothing. The hub is kept and the error names `refs/lym/heads/main-weft` with the same commands. OK.
3. Push declined outright (`pre-receive`, exit 1 with a message, no kill):
   - Board ref: the error is "origin took the records branch, but its hooks or rules declined the board ref", and the log shows the records branch created, then deleted. OK.
   - Records ref: the error is the host-refusal way forward, "use a host that accepts refs outside refs/heads/ and refs/tags/". OK.
4. Control: a hook that logs and lets both pushes finish seeds a hub. OK.
5. A hook that kills the client and then sleeps does not open a window for a read before the ref lands.
   `gitexec` waits for the stderr pipe that the hook's `receive-pack` holds, so lym's read of origin happens after the hook returned and finds the ref.
   The "origin does not hold the pushed ref yet" branch of `withdrawUnansweredSeedPush` is reached only by the kill-and-decline drive in 2, which keeps the hub.
6. Seeding lym killed with `kill -9` while a `pre-receive` sleeps on one ref:
   - Records ref: 12 seconds later origin holds `refs/lym/heads/main-weft` and no board, and the hub directory is there with its prime records branch, no `branch.<b>.merge` keys and no registration in the sandbox list.
   - Board ref: origin holds both refs and the hub directory is there.
   - In both, `fabric status`, `fabric pairs` and `fabric cleanup --remote` answer without changing origin. `board sync` refuses with "has no upstream in a one-repository hub, so it was never seeded onto origin … run the same `lym fabric clone --single-repo` again with `--reset`".
   - `board upsert` of a note succeeds and writes locally. See Rejected for the wording.
7. Prime records branch key, prime pair on `alt` by plain git, `lym fabric cleanup --remote` as a dry run:
   - Unset or empty: refused with the `git config` way forward.
   - `main-weft` padded with spaces: accepted, `main` and `main-weft` protected.
   - `nope-weft`, `alt`, `main-weft-weft`: refused, "holds no branch of that name".
   - `main`: refused, "is not a records branch name".
   - `alt-weft`, the records branch of the branch the prime is on now: accepted, and `main-weft` is listed with no `protected` flag. See Rejected.
   - `main-weft`: accepted, `main` and `main-weft` protected.
   - `lym fabric prune` as a dry run lists nothing in every row.
   - `grep` for a `"-weft"` literal in production Go files outside `internal/weftname` finds comments only.
8. Faulted records store, config restored from a copy between rows, `board upsert`, `board sync`, `fabric status`, `fabric cleanup --remote` after each:
   - Fetch refspec unset; fetch refspec the code refspec; the `origin` remote removed: `board sync` and `cleanup --remote` refuse with the fetch-refspec or `remote add` way forward, and origin does not move.
   - Code fetch refspec, no push refspec, no prime key, merge settings left (`branch.main.merge`, and `branch.main-weft.merge`): refused, origin unmoved.
   - The same with origin's code default branch renamed `trunk`: refused, origin unmoved.
   - A merge setting on `branch.feature.merge` only: refused on the first run of this row. The re-run on a hub seeded by the `.dev-bin` lym is refused through `branch.main-weft.merge`.
   - `fabric status` answers in every row and never touches origin.
   - Control: a hub made by `lym harden sandbox new` (two repositories) answers `fabric status` and `cleanup --remote` as two repositories.
9. Clean seed on a hub made by the `.dev-bin` lym: the board's `@{u}` is `refs/remotes/origin/main` and the records branch's is `refs/remotes/origin/main-weft`, with `[ahead 4]` and `[ahead 1]`, each ahead of an origin that is not behind.
   The board's ahead count is the clone's commits that the detached sync pushes later.

An early set of drives (probe 7, a first run of probe 8, and the first clean seed) ran on a hub the lym on the path seeded, which is an older build.
Its records branch had no upstream.
Probe 9 and the second run of probe 8 were repeated on a hub the `.dev-bin` lym seeded, and the results above are from those.

Tests read, not run, since this session may not edit the tree: `clone_singlerepo_killedpush_integration_test.go` has four rows that cover the withdrawal (post-receive rows assert an empty `refs/lym/` and no hub) and the read before it (pre-receive rows assert a kept hub, the error text and, for the board, the records branch left on origin).
A removed withdrawal leaves refs on origin in the two post-receive rows, and a removed read rolls back a hub the pre-receive rows expect kept; no sabotage run was made.
The `OnlyABranchMergeSettingLeft` row in `TestOriginNamespaceAt` and the `OneRepositoryKeyNamesTheBoardBranch` row in `TestCleanup_RefusesWhenPrimaryWeftBranchIsUndeterminable` fail without their check by construction of the faulted store they build.

## Rejected

- The clone's way forward after a killed seeding lym is circular.
  `lym board sync` says the board "was never seeded onto origin" and names `lym fabric clone --single-repo … --reset`, in `internal/fabricengine/pushclassified.go`.
  After the kill in probe 6, origin holds the whole seed, so the sentence is false, and a one-URL clone onto an origin that already holds a namespaced board is refused.
  The file is not in the round's changes list, and the finding is not BLOCKING, so it is not raised.
  The `--reset` path could not be driven, since `lym fabric clone` is refused to a task session ("lym fabric clone is refused from the task session …").
  The next round should settle it with a verb a task session may run.
- A seed that took the records branch and removed it again says nothing of it.
  The killed-push rows A and B return "the seed push of the board failed … run the same clone again" without saying the records branch was on origin and was withdrawn.
  The declined-board message does say it.
  The text is a LOW wording gap and not raised.
- A prime key naming the records branch of the branch the prime is on now (`alt-weft`) is accepted and leaves `main-weft` open to the sweep.
  The `primaryWeftBranch` comment and both refusals state that fabric cannot tell this from the right value, so it is the documented residual and not a defect.
- A records store with every mark removed (no push refspec, no prime key, no `branch.*.merge`) and the code fetch refspec left reads as two repositories.
  With origin's default branch renamed `trunk` the `board sync` run then created `refs/heads/main` on origin, a records ref outside `refs/lym/`.
  The fetch refspec of such a store is the one a two-repository records store has, so no setting tells them apart.
  Reaching it takes four deleted settings, and it is outside what the fix text for F8 promised.
- `seedPushOutcomeUnknown` treats a stalled git (`Stalled`) as an answer that left origin as it was.
  A stall is the low-speed bound on a remote transport, which cannot be driven on a local bare repository, so this stays unverified and not raised.
- A killed push onto an origin that already held a records branch (`seed.observedTip` non-empty) restores through `restoreSeededRecordsBranch`'s update path.
  No sandbox origin can hold one before the seed, since `sandbox new` seeds from a fixed template, and `killedpush` holds no row with an observed tip.
  Could not verify.
- A kill-and-sleep hook that lets origin apply a push after lym has read it could not be built with a local transport; see probe 5.

## Coverage

Not driven: the joined clone, the archive tags, two clones of one origin, lym's own remote delete and a pair verb killed mid-operation need a verb a task session may not run (`fabric clone`, `add`, `remove`, `push`, `pull`, `commit`, `checkout`, `cleanup --apply`).
The refusal text is above.

Teardown check, run at the end:

```
lym harden sandbox clear   -> {"ended":[],"ok":true,"root":".../.lym/harden-sandbox"}
pgrep -af "harden-sandbox" -> lists only another run's processes (a konsole, a tmux attach and a claude session of the worktree
                              harden-sandbox-fabric-verbs) and this check's own shell; no process opens this worktree's sandbox root
tmux ls | grep harden-sandbox -> the session harden-sandbox-fabric-verbs, which belongs to that other run
ls .lym/harden-sandbox     -> hubs.json hubs.json.lock
git for-each-ref refs/lym refs/tags -> no refs/lym ref; the refs/tags/archive* tags were in the repository before this round
lym harden sandbox list    -> {"hubs":[],"ok":true}
```

The scratchpad holds the hook templates, scripts and logs of this round.
It is outside the worktree and holds no hub.

## Focus departures

- "run the same hook with the lym on the path, built before commit 704c7ae": the comparison binary is built from `git archive 704c7aea6~1` in the scratchpad and run by path, not put on `PATH`, because the seed runs in the process of whichever binary runs `harden sandbox new`.
- "Read the tests e9d677c added and check that each fails when the withdrawal or the read before it is removed": the tests were read and not run against a mutated tree, since the review may not change a source file.
