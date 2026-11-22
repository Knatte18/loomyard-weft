---
round: 2
focus:
  - |
    Seed rollback when the board push ends without a definite answer.
    Site: `internal/fabricengine/destroy.go`, `pushCloneSeed` and its fail path, `restoreSeededRecordsBranch`, `checkSeedPush`, `seedPushCause`.
    Question: when the seed's board push to `refs/lym/heads/main` fails by a killed client, a dropped connection or a timeout, rather than by a rejection,
    can origin still apply the board after the seed has restored the records branch and removed the hub?
    The end state to look for is origin holding `refs/lym/heads/main` with no `refs/lym/heads/<records-branch>` and no hub.
    Repro: plant a `pre-receive` in the seed's bare origin that sleeps on `refs/lym/heads/main`, kill the client `git push` mid-sleep,
    then wait for the hook to return and list origin with `git ls-remote`.
    Keep the sandbox origin alive past the hook's return, so the last step is watched and not inferred from a plain-git control.
    Also run a hook that rejects the board ref, one that rejects the records ref, and one that sleeps but lets the push finish; check the hub directory and origin after each.
    Sabotage the new handling and watch the scenario fail, so the drive is shown to reach it.
  - |
    Layout detection under a faulted records config.
    Site: `internal/fabricengine/recordslayout.go`, `originNamespaceAt`, `OriginRefNamesAt`, `readOriginLayout`; every caller that hands the namespace to a push, a fetch, a listing or a delete.
    Question: on a one-repository hub whose records store keeps its push refspecs, `branch.main.merge refs/lym/heads/main` and `lym.primeRecordsBranch`,
    is a store with `remote.origin.fetch` removed, set to `+refs/heads/*:refs/remotes/origin/*`, or with no `origin` remote or an empty config, ever read as two repositories?
    Check that a board write, `lym fabric status`, `cleanup --remote` as a dry run and `lym board sync` each either keep to `refs/lym/` or stop with an error that names the way forward.
    Check that a healthy two-repository store and a path with no store still read as two repositories, and that the answer is not cached across a change within one process.
    Repro: edit the config in turn, restore it from a copy between rows, and run `git ls-remote` on origin after each verb.
    Sabotage the check by restoring the old read and watch the drive fail.
  - |
    The prime records branch key and the orphan sweep.
    Site: `internal/fabricengine/cleanup.go`, `primaryWeftBranch` and its one-repository branch; `cleanup --remote` and `prune` as dry runs.
    Question: with the prime pair on another branch, does the sweep list the prime's records branch as deletable under any value of `lym.primeRecordsBranch`:
    unset, empty, padded with spaces, naming a branch that does not exist, naming a branch that exists but is not the prime's, or naming the right one?
    Check that each fail-closed refusal names a `git config` way forward and that a reached test covers it.
    Repeat on a joined clone if a join can be made; otherwise record that it could not be verified, with the refusal text.
  - |
    The seed's refusal wording and the seeded records branch's upstream.
    Site: `internal/fabricengine/destroy.go`, `seedRefusedByOrigin`, `seedPushCause` and the upstream settings written after the board lands.
    Question: when origin has just accepted the records ref and a hook declines only the board ref, does the message name the real cause and a way forward that fits it?
    When the origin refuses the records ref, does the message still name the host-refusal way forward?
    After a clean seed, does `git rev-parse @{u}` answer on both the board and the seeded records branch, and does `git status -sb` show a correct ahead and behind count on each?
  - |
    The previous round's fixes are unreviewed text.
    Re-drive the fixes of the four entries above against a fresh `lym harden sandbox new --single-repo`, built from this tree, and read each fix as new code.
    Check each fix's tests for a case that fails when the fix is removed.
  - |
    Regions no round has driven on the substrate.
    Question: which of the following scenarios can be driven, and what do they show about `refs/lym/` against plain `refs/heads/` and `refs/tags/` on origin?
    Drive each that the session's verb permissions allow; for a scenario a refused verb blocks, record the refusal text and report it as could not verify.
    The scenarios: the archive tags of a discarded task branch, from the code store and from the records store, and the ref each lands under;
    the join of a seeded origin and the config it writes; the refusals of `--single-repo` before anything is created;
    two clones of one origin each pushing and writing to the board, with a sequential control and a second sandbox hub running the same drive;
    the remote delete of a missing and a present namespaced branch, and its idempotence;
    a pair verb killed mid-operation; origin variations (`file://`, a trailing `.git`, a HEAD that is not `main`).
  - |
    Safety pass over what the first round could not drive.
    Question: with the items above settled, what does a clean-room pass over the one-repository layout find that a first pass missed?
    Look at crash and rebirth of a seed that was killed before the seed push, a leftover hub of a killed seed that no listing names,
    and board commits made by the start walk of a verb that nothing pushes.
    The round invents no work when it finds none.
---

## Round context

Round 2 closes the residual of round 1 and sweeps the regions round 1 could not drive.
The reviewer model must differ from round 1's, which was `opus`.

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
Use the `.dev-bin` lym for hub verbs, and the lym on the path for `harden sandbox` verbs.
