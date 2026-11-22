---
round: 3
focus:
  - |
    Seed rollback when the records push or the board push ends without a definite answer.
    Site: `internal/fabricengine/destroy.go`, `pushCloneSeed`, the helper that withdraws an unanswered push, `seedPushOutcomeUnknown`,
    `restoreSeededRecordsBranch`; commit 704c7ae changed it to read origin before a rollback, and e9d677c added the killed-push tests.
    Question: when the seed's push of `refs/lym/heads/<records-branch>` or of `refs/lym/heads/main` is killed, drops or times out after origin received it,
    does the seed read origin before it removes the hub, and does every end state leave either an empty `refs/lym/` with no hub
    or a kept hub whose error names each ref origin may still take with the `ls-remote` and `push --delete` commands that clear it?
    Also check that a push origin rejects outright is rolled back and the hub removed, and that a failed read or a failed restore keeps the hub.
    Repro: plant a `pre-receive` or `post-receive` in the fresh bare origin through `GIT_TEMPLATE_DIR` that sleeps on one ref, kills the client once the ref is applied,
    or kills the client and exits non-zero; after the hook returns, list origin with `git ls-remote`, and look at the hub directory.
    Drive both refs, and a hook that lets the push finish as a control.
    Show that each drive reached the mechanism: run the same hook with the lym on the path, built before commit 704c7ae, and watch the end state differ.
    Read the tests e9d677c added and check that each fails when the withdrawal or the read before it is removed.
    Check that `pattern/PATTERN-single-repo-hub.md`, section The clone, states what the code keeps for a rejected push, an unanswered push and a kept hub.
  - |
    The prime records branch key and the orphan sweep.
    Site: `internal/fabricengine/cleanup.go`, `primaryWeftBranch` and both of its refusals; commit ba00dc2 renders the clone-time records branch through `RecordsBranchName`.
    Question: with the prime pair on another branch, is the prime's records branch ever listed as deletable under a value of `lym.primeRecordsBranch` that is unset, empty,
    padded, a missing branch, the records branch of the branch the prime is on now, the board branch, or the right one?
    Check that every refusal names a `git config` way forward that cannot be read as the branch the prime is on now, spells no suffix literally,
    and that `TestCleanup_RefusesWhenPrimaryWeftBranchIsUndeterminable` has a row for each refused value that fails without its check.
    Repro: move the code store to another branch and the records store to its records branch by plain git, set the key by `git config`,
    run `lym fabric cleanup --remote` and `lym fabric prune` as dry runs, and read what each lists.
    Check by grep that no production file outside `internal/weftname` spells the records suffix as a string literal.
  - |
    Layout detection under a faulted records config.
    Site: `internal/fabricengine/recordslayout.go`, `recordsStoreClaim`, `originNamespaceAt`, `OriginRefNamesAt`; commit 704c7ae made the claim read a branch's merge setting.
    Question: on a one-repository hub, is a records store that lost its namespaced fetch refspec, its push refspecs and `lym.primeRecordsBranch`
    but kept its origin and a `branch.<b>.merge` value under `refs/lym/` still read as two repositories by any caller that pushes, fetches, lists or deletes on origin?
    Include a merge setting on a branch other than `main`, and a store whose only mark is a merge setting.
    Check that `lym board sync`, `lym board upsert`, `lym fabric status` and `lym fabric cleanup --remote` each keep to `refs/lym/` or stop with an error that names a way forward.
    Repro: edit the records store's config in turn, restore it from a copy between rows, name origin's code default branch something other than `main` in one row,
    and list origin with `git ls-remote` after each verb.
    Check that a healthy two-repository store and a path with no store still read as two repositories, and that a test row fails when the merge read is removed.
  - |
    The previous round's fixes as new text.
    Re-drive on a fresh `lym harden sandbox new --single-repo` built from this tree: a hook declining only the board ref names origin's hooks or rules as the cause,
    a hook declining the records ref names the host-refusal way forward, and a clean seed answers `@{u}` on the board and the seeded records branch with a correct ahead and behind count.
    Read each fix as code and check that its test fails when the fix is removed.
  - |
    Safety pass over a killed seeding lym.
    Question: when the seeding `lym` itself is killed while its push waits in a hook, what do origin and the hub directory hold, and do the verbs a reader can run
    (`lym fabric status`, `lym fabric pairs`, `lym fabric cleanup --remote`, `lym board sync`, `lym board upsert`) answer in a way that names the way forward?
    Check the wording at `internal/fabricengine/pushclassified.go` that says a board with no upstream was never seeded onto origin, against an origin that holds the whole seed.
    Check that the message of a seed that took the records branch and then removed it again says so.
  - |
    Regions that need a verb the task session refuses.
    The verbs `fabric clone`, `add`, `remove`, `push`, `pull`, `sync`, `commit`, `reconcile`, `checkout`, `cleanup --apply` and `prune --apply` answer a refusal in a task session.
    Record the refusal text again for a verb a drive needs, report the region as could not verify, and drive only what the permitted verbs reach.
    The round invents no work where no permitted verb reaches the region: the joined clone, the archive tags, two clones of one origin, lym's own remote delete and a pair verb killed mid-operation.
---

## Round context

Round 3 closes the residual of round 2 and re-drives its three fixes, which are new text no fresh reviewer has seen.
The reviewer model must differ from round 2's, which was `fable`.
Commit 8a8ff40 only reshaped a comment and e9d677c only added tests; 704c7ae and ba00dc2 carry the code.

## Standing charter

The modules are `internal/fabricengine`, `internal/fabriccli` and `internal/gitrepo`.
The operator's focus binds this campaign: only the one-repository hub layout that commit 57015a52 added.
Origin holds the code under `refs/heads/` and `refs/tags/`, and the records and the board under `refs/lym/heads/` and `refs/lym/tags/`.
Everything else in fabric, the two-repository layout included, is out of scope, and no round steers outside this focus.

### What to read

- `pattern/PATTERN-single-repo-hub.md`, the `PATTERN-code-conventions.md` entry on lym-owned records-namespace branches, `README.md` and `docs/getting-started.md` on the layout and its hazards.
- `internal/fabricengine/doc.go`, `internal/fabriccli/doc.go` and `internal/gitrepo/doc.go`.
- The code paths: `recordslayout.go`, `clone.go`, `destroy.go` and `cleanup.go` in `internal/fabricengine`, and every site that hands the namespace to a push, a listing, a fetch or a delete.
  `grep -rn "originNamespaceAt\|RefNamespace\|refs/lym\|ns\.Branch\|ns\.Tag" internal --include=*.go` enumerates them.
- `internal/fabriccli/clone.go` and `fabric.go`; in `internal/gitrepo`, `refnamespace.go`, `push.go`, `pushdestination.go`, `pushretry.go`, `remote.go`, `refreads.go`, `blobread.go`, the tolerant config read and `merge.go`; `internal/hubforge/hub.go`.
- `lym fabric clone --help` and the `--help` of the other fabric verbs; `lym harden sandbox --help`.
- The existing tests, for what they leave undriven. Every live test of these modules is tier 1 or below; none of them is the review.

### Layouts

Drive the one-repository hub built by `lym harden sandbox new --single-repo`.
The two-repository hub that `lym harden sandbox new` builds is out of scope and is built only as a control, recorded as one.
Scenarios that need another origin use bare repositories and containers made under the sandbox root, cloned with the `.dev-bin` lym the sandbox prints.
A verb refused to a task session is recorded with its refusal text and reported as could not verify.

### Out of scope

- The two-repository layout and every behaviour that holds only in it.
- The rest of fabric: junction wiring, anchors, merge conflict handling, the dobby, the board's verbs, reed and every module outside the three named.
- Conversion of the earlier one-repository layout (`lym-bolt`): a refusal that names the cleanup is the correct result.
- A host that refuses refs outside `refs/heads/` and `refs/tags/`: the seed's refusal that names a two-repository hub is correct.
- GitHub-owned refs and a plan-gated rules API: the sandbox has no GitHub remote.
- Correct absences: a code store holds no `refs/remotes/origin/*-weft` and no `refs/lym/` ref; origin's `refs/heads/` holds code branches only; a joined hub writes no `lym.primeRecordsBranch` into the code store.
- A timeout under the concurrent copies' CPU peg, and a rerun of the existing tests as a campaign item.
- The live gate's package-path setup in `internal/loomcli/hardenwiring.go`, outside the modules.

### Live-substrate cost declaration

No live test or verb of these modules spawns a real LLM session.
Every drive is a plain `git` and `lym` process on local bare repositories, so the campaign names no tier-3 test and bans none.
A fabric verb that starts a reed server or a tmux pane for the sandbox hub belongs to the teardown check.

### Teardown check

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

Zero stray state means: `clear` ends nothing it did not already list, no process or tmux session names the root,
the root holds no hub, bare origin or container directory, the worktree's own repository holds no ref the round pushed, and `lym harden sandbox list` lists no hub.
Use the `.dev-bin` lym for hub verbs, and the lym on the path for `harden sandbox` verbs, except that a seed is made in process by the binary that runs `harden sandbox new`,
so a seed under review runs as `.dev-bin/lym harden sandbox new`.
