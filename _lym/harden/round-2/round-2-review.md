---
verdict: BLOCKING
findings:
  - id: F6
    carries: F1
    severity: BLOCKING
    class: design
    location: internal/fabricengine/destroy.go#pushCloneSeed
    summary: "The board half of F1 is fixed, but the records push has the same fault: when the seed's push of refs/lym/heads/<records-branch> ends without origin's answer (client killed in S6, exit -1), pushCloneSeed removes the hub without reading or restoring origin, and origin then applies the records branch, so a failed seed leaves refs/lym/heads/main-weft behind with no hub; CONFIRMED in S6. The board's kept-hub path and this records path have no test, and pattern/PATTERN-single-repo-hub.md still states that a failed seed leaves nothing behind."
    fix: "In pushCloneSeed (internal/fabricengine/destroy.go:1754-1756), handle a failed records push as the board push is handled: when seedPushOutcomeUnknown(err) holds, read origin's refs/lym/heads/<seed.recordsBranch> with one ls-remote before any teardown. A ref at recordsTip is returned to seed.observedTip with restoreSeededRecordsBranch(seed, recordsTip) and the hub is then removed. A ref origin does not list while the outcome is unknown, a failed read, or a failed restore keeps the hub and returns the keep error naming the ref and the same ls-remote and push --delete commands. Generalise withdrawUnansweredBoard to take the branch, its pushed tip and the act that withdraws it, so both pushes share one path. Add two tests in clone_singlerepo_killedpush_integration_test.go: one whose post-receive kills the client once refs/lym/heads/<records-branch> is updated, asserting an empty refs/lym/ and no hub; one whose pre-receive kills the client on the board ref and then exits non-zero, so origin never lists the board, asserting the hub is kept, the error names both refs and origin still holds the records branch. In pattern/PATTERN-single-repo-hub.md, section The clone, replace the sentence that a failed seed removes what it pushed and leaves nothing behind with the rule the code keeps: a rejected push is rolled back and the hub removed, a push that ended without origin's answer is read back first, and a ref origin may still take keeps the hub with a way forward naming the refs; name the new tests there."
  - id: F7
    severity: BLOCKING
    class: consistency
    location: internal/fabricengine/cleanup.go:369
    summary: "The refusal the F3 fix added spells the records suffix as the literal -weft in its way forward, against PATTERN-code-conventions' named-constant rule and branchname.go's rule that code outside weftname never spells it; its placeholder <prime-branch>-weft reads as the branch the prime is on now, and the new check accepts any existing branch, so with the prime on alt a key of alt-weft, or of the board branch main, passes and the sweep lists main-weft with no protection; CONFIRMED by dry run in K."
    fix: "In primaryWeftBranch (internal/fabricengine/cleanup.go:357-371), build the way-forward tail once and use it in both refusals, the one at line 360 and the new one at line 369. Render the suffix from weftname.Suffix through RecordsBranchName, never as a literal, and name the value unambiguously: the records branch of the code branch this hub was cloned on, not of the branch the prime is on now, for example RecordsBranchName(\"<clone-time-code-branch>\"). After the BranchExists check, also refuse a key that WeftWarpSlug does not accept, such as the board branch, with the same way forward. Add a row to TestCleanup_RefusesWhenPrimaryWeftBranchIsUndeterminable for a key naming OneRepositoryBoardBranch, run with the prime pair on another branch."
  - id: F8
    carries: F2
    severity: LOW
    class: design
    location: internal/fabricengine/recordslayout.go#recordsStoreClaim
    summary: "The F2 fix reads two of the three marks F2 named: a store that lost its namespaced fetch refspec, its push refspecs and lym.primeRecordsBranch but kept its origin and branch.main.merge refs/lym/heads/main still reads as two repositories; with origin's code default branch not main, lym board sync answered ok and created the board at plain refs/heads/main on origin, a records ref outside refs/lym/; CONFIRMED in LK. A store with an empty config also still reads as two repositories, but has no origin to push to."
    fix: "In recordsStoreClaim (internal/fabricengine/recordslayout.go:110-133), extend the one git config --get-regexp listing to every branch.<name>.merge key, and return a claim naming the key and value when a value starts with RecordsNamespaceRoot, so originNamespaceAt answers its existing error. Add a row to TestOriginNamespaceAt's faulted-store table: a store with the code fetch refspec, no push refspec and no prime key, whose branch.main.merge is refs/lym/heads/main, wanting the fetch-refspec way forward."
prior:
  - id: F1
    status: open
    answer: "The board push is fixed and holds on the substrate (S4 with its pre-fix control S4c, S5, S8). The same unanswered outcome on the records push is still rolled back blind (S6); F6 carries it."
  - id: F2
    status: open
    answer: "Fixed for a store that keeps its push refspecs or its prime key (L with the pre-fix control Lc). The third mark F2 named, branch.<b>.merge, is not read, and a store keeping only that mark leaked the board to plain refs/heads/main (LK); F8 carries it at LOW."
  - id: F3
    status: resolved
    answer: "A key naming a missing branch now refuses with a git config way forward (K); the neighbouring gaps at the same hunk are F7."
  - id: F4
    status: resolved
    answer: "A hook declining the board is named as origin's hooks or rules; the pre-fix control S1c still shows the host wording, and a declined records ref keeps it (S2)."
  - id: F5
    status: resolved
    answer: "A hub seeded by the .dev-bin lym answers main-weft@{u} with refs/remotes/origin/main-weft and shows ahead 1; the hub the pre-fix lym seeded has no such upstream (P1)."
---

The review holds the method, the probe log, the focus walk, one block per finding, the observations below the incremental bar,
the deferred item, the teardown check and the focus departures.
I formed every finding before opening round 1's review and fixer report.

## Method

`D` is `/home/knatte/Code/loomyard-LYMHUB/harden-fabricengine/.dev-bin/lym`, built from HEAD `c25e17ac2` and holding the fix text.
`P` is the lym on the path, `/home/knatte/go/bin/lym`, built at `2dbfdc314` (`go version -m`), the commit before the fixes.
`S` is `.lym/harden-sandbox`, the sandbox root.

- **P0.** `D fabric clone --single-repo --shortname cl --into <c> <bare>` is refused:
  `lym fabric clone is refused from the task session ly:harden-fabricengine:harden-burler-review; build a scratch hub as a hubforge test fixture instead, and when the work itself needs this verb, stop and report status: FAILED so the run ends stuck and the orch runs it`.
- **Seeds.** Every seed is one `lym harden sandbox new --single-repo`, which clones in process with the binary that runs it.
  `D harden sandbox new` therefore drives the fixed seed, and `P harden sandbox new` the pre-fix seed, which is the sabotage control.
- **Hooks.** `GIT_TEMPLATE_DIR=S/r2/tpl` plants a logging `pre-receive` and `post-receive` in the fresh bare origin at its `git init`.
  `R2_REJECTREF` declines a ref, `R2_SLEEPREF` and `R2_SLEEPSECS` sleep on one, `R2_KILLREF` kills the client once origin applied it.
  Each hook logs to a file outside the hub container, so the log outlives the container the sandbox removes on a failed seed.
- **Watching origin past the hook.** The sandbox removes its container when the seed fails.
  To watch origin after the hook returns, the driver stops the seeding lym with SIGSTOP, lists origin by hand, then continues it:
  in S6 at the kill, and in S5 at lym's own read of the board, through a `remote.origin.uploadpack` wrapper that stops lym before origin answers.
  The read lym acts on is the one origin gave before the hook returned, so the stop does not change lym's decision.
- **Probe log.** I kept the log in `.scratch/probe-log.md` while driving and folded it in here,
  because the round ends when this file exists.

## Probe log

### Seed, board push (focus 1, 4, 5)

- **P1, clean seed.** `D harden sandbox new --single-repo` made `hub-2360074276`.
  Hook log: `refs/lym/heads/main-weft` applied first, `refs/lym/heads/main` second.
  Records store: `branch.main.merge refs/lym/heads/main`, `branch.main-weft.merge refs/lym/heads/main-weft`.
  `git rev-parse --symbolic-full-name main-weft@{u} main@{u}` answers `refs/remotes/origin/main-weft` and `refs/remotes/origin/main`.
  `git status -sb` reads `## main-weft...origin/main-weft [ahead 1]` in the records store and `## main...origin/main` in `_bolt`;
  `git rev-list --left-right --count main-weft...origin/main-weft` answers `1 0`, the sandbox's own commit.
  Control: `hub-1569252615`, seeded by `P`, has no `branch.main-weft.*`.
- **S1, hook declines the board.** `D`: `origin took the records branch, but its hooks or rules declined the board ref refs/lym/heads/main: ... ! [remote rejected] ... (pre-receive hook declined) ...; way forward: fix or lift the rule that declines refs/lym/heads/main, then run the same clone again`.
  Hook log: `main-weft` applied, the board declined, `main-weft` deleted at its pushed tip; origin's refs afterwards are `refs/heads/main` alone.
- **S1c, control.** `P`, same hook: `origin refused the seed push of the board: ...; way forward: use a host that accepts refs outside refs/heads/ and refs/tags/, or make a two-repository hub ...`.
- **S2, hook declines the records ref.** `D`: `origin refused the seed push of the records branch: ...; way forward: use a host that accepts refs outside refs/heads/ and refs/tags/, or make a two-repository hub with lym fabric clone <records-url> <code-url>`.
  Hook log: one line, the declined ref; origin took nothing.
- **S3, hook sleeps 5 s on the board and the push finishes.** `D`: `ok: true`, both refs applied, the board after the sleep.
- **S4, client killed after origin applied the board.** `post-receive` sends SIGKILL to the client `git push` once `refs/lym/heads/main` is updated.
  `D`: `the seed push of the board failed: ... exit -1; way forward: run the same clone again`.
  Hook log: board applied, client killed, then a push deleting `refs/lym/heads/main` at the seed's tip, then one deleting `refs/lym/heads/main-weft`;
  origin's refs afterwards are `refs/heads/main` alone.
- **S4c, control.** `P`, same hook: after the kill the log holds only the delete of `main-weft`, and origin ends as
  `refs/heads/main` and `refs/lym/heads/main`: a board with no records branch and no hub, round 1's F1 end state.
- **S5, client killed while the hook sleeps 20 s on the board.** Client killed at 12:14:42.04.
  lym read origin's board at 12:14:52.07, ten seconds later (gitexec's wait on the dead git's pipes), and was stopped there.
  The hook woke at 12:15:01.98; `git ls-remote` at 12:15:05 listed `refs/lym/heads/main` at the seed's tip `b89323e` and `refs/lym/heads/main-weft`,
  and the hub directory held `_bolt`, the code store and the records store.
  Continued, lym answered: `the seed push of the board ended without origin's answer: ... exit -1; origin does not list the board yet, and may still take it, so the hub at <hub> is kept and origin may hold refs/lym/heads/main and refs/lym/heads/main-weft; way forward: list them with git -C <store> ls-remote origin refs/lym/heads/main refs/lym/heads/main-weft, delete each one listed with git -C <store> push origin --delete <ref>, then remove the hub at <hub> and run the same clone again`.
  The hook log holds no push after the board's: lym restored nothing, so origin never held a board without its records branch.
- **WF, the kept hub's way forward by hand.** On S3's hub, `git -C <store> ls-remote origin refs/lym/heads/main refs/lym/heads/main-weft` listed both,
  and `git -C <store> push origin --delete <ref>` deleted each; origin then held `refs/heads/main` alone.
- **S8, the board push outlives the remote deadline.** Hook sleeps 700 s on the board.
  `D` answered at ten minutes: `the seed push of the board ended without origin's answer: ... timed out after 10m0s; origin does not list the board yet, and may still take it, so the hub at <hub> is kept ...`.
  The hook never woke: the deadline's group kill ended the local receive-pack with it.

### Seed, records push (focus 1, improvised)

- **S6, client killed while the hook sleeps 8 s on the records ref.** lym stopped and the client killed at 12:15:42.59.
  The hook woke at 12:15:50.57; `git ls-remote` at 12:15:53 listed `refs/lym/heads/main-weft` at the seed's tip `1842a31` and no board.
  Continued, lym answered: `the seed push of the records branch failed: ... exit -1; way forward: run the same clone again`.
  The hook log holds no push and no read after the woken one: lym neither restored the branch nor looked at origin, and it removed the hub (F6).

### Killed lym (focus 7)

- **S9, the seeding lym killed while its board push waits in the hook.** The `git push` child outlived lym.
  Origin ended with `refs/lym/heads/main` and `refs/lym/heads/main-weft`, the whole seed.
  The hub stays on disk with no `branch.*` upstream and no junction, and `lym harden sandbox list` does not name it.
  In it, `D fabric status` and `D fabric cleanup --remote` answer `ok: true`, and `D fabric pairs` reports `warp _lym junction missing`.
  `D board upsert` commits locally, and `D board sync` refuses:
  `the board at <hub>/_bolt has no upstream in a one-repository hub, so it was never seeded onto origin and is not pushed; way forward: run the same lym fabric clone --single-repo again with --reset`.
  `_bolt` then holds three commits origin lacks: `lym: seed stencils`, `lym: seed specs` and the probe's `board sync`.

### Layout detection (focus 2)

`faults.sh` restores the records store's config from a copy between rows and lists origin after every verb.
The verbs are `fabric status`, `fabric cleanup --remote`, `fabric prune`, `board sync`, `board upsert` of a new note and `board sync` again.

| Row | Records store | `D` result |
|---|---|---|
| L-healthy | untouched | all `ok: true`; the note lands at `refs/lym/heads/main`; `refs/heads/main` unmoved |
| L-unset | `remote.origin.fetch` unset | `status`, `prune`, `upsert` ok; `cleanup --remote` and `board sync` refuse (text below); origin unmoved |
| L-code | `remote.origin.fetch` = `+refs/heads/*:refs/remotes/origin/*` | the same as L-unset |
| L-noorigin | `git remote remove origin` (only `lym.primerecordsbranch` is left) | `cleanup --remote` and `board sync` refuse, naming `lym.primeRecordsBranch main-weft` and `git -C <store> remote add --no-tags origin <code-url>`; origin unmoved |
| L-empty | config truncated to empty | `cleanup --remote` ok, read as two repositories; `board sync` runs `git push origin refs/heads/main:refs/heads/main` and fails, `'origin' does not appear to be a git repository`; origin unmoved |
| Lc, control | `P` on `hub-1569252615`, fetch unset | `cleanup --remote` ok; `board sync` pushes plain `refs/heads/main`, rejected three times as a non-fast-forward |
| C2, control | `D harden sandbox new`, a two-repository hub | `cleanup --remote` ok, the note lands at the records bare's `refs/heads/main` |

L-unset's refusal: `the repository at <store> holds the push refspec refs/heads/*:refs/lym/heads/*, so it is a one-repository records store, but no origin fetch refspec maps refs/lym/heads/; way forward: restore its fetch refspecs with git -C <store> config --replace-all remote.origin.fetch '+refs/lym/heads/*:refs/remotes/origin/*' and git -C <store> config --add remote.origin.fetch '+refs/lym/tags/*:refs/tags/*'`.
With the config restored, `D board sync` answered `ok: true` and origin's board took every probe note.

- **LK, a store keeping only its merge setting.** On `hub-709752712`: fetch set to the code refspec, both push refspecs and the prime key removed,
  `remote.origin.url` and `branch.main.merge refs/lym/heads/main` kept.
  `D fabric cleanup --remote` answers `ok: true`, read as two repositories.
  (a) `D board sync` is rejected against origin's plain `main`, and its way forward offers `git reset --keep origin/main` and `git merge origin/main` in `_bolt`.
  (b) After `git branch -m main trunk` in the bare, `D board sync` answers `ok: true` and origin gains `refs/heads/main` at `650c5fd`, the board (F8).

### Prime records key (focus 3)

`D fabric checkout alt` is refused with the task-session text, so the prime pair was moved by hand: the code store to `alt`, the records store to `alt-weft`.
Each row sets the key, then runs `D fabric cleanup --remote` and `D fabric prune`, both dry runs.
`prune` listed nothing in every row.

| Key | `cleanup --remote` |
|---|---|
| `main-weft` | `main-weft` protected |
| unset | refused: `the records store at <store> has no lym.primeRecordsBranch; refusing to enumerate orphan weft branches — set it with git -C <store> config lym.primeRecordsBranch to <prime-branch>-weft` |
| empty | the same refusal |
| `  main-weft  ` | `main-weft` protected |
| `nosuch-weft` | refused: `the records store at <store> names "nosuch-weft" in lym.primeRecordsBranch, but holds no branch of that name; refusing to enumerate orphan weft branches — set it with ... to <prime-branch>-weft` |
| `alt-weft` | `ok: true`; `main-weft` listed with no `protected` |
| `main` | `ok: true`; `main-weft` listed with no `protected` |

### Refused verbs (focus 6)

From `hub-2360074276`'s prime, each of these answered the P0 refusal with its own verb name:
`fabric add`, `remove`, `push`, `pull`, `sync`, `commit`, `reconcile`, `checkout`, `cleanup --apply --remote`, `prune --apply` and `clone` in its join form.
`fabric status`, `list`, `pairs`, dry `cleanup` and `prune`, and `board upsert`, `sync`, `list`, `labels` ran.

## Focus walk

| # | Entry | Status | Evidence |
|---|---|---|---|
| 1 | Seed rollback when the board push ends without a definite answer | FAIL | The instance to judge is not a defect any more: a landed board is withdrawn before the restore (S4, control S4c), and an unlisted one keeps the hub (S5, S8). The records push has the same fault unhandled (S6, F6). |
| 2 | Layout detection under a faulted records config | WARN | A store keeping its push refspecs or prime key errors with a way forward on every writing verb (L, control Lc); a healthy two-repository store reads as two (C2). An empty config reads as two repositories and cannot push (L-empty). A store keeping only `branch.main.merge` leaks (LK, F8). Caching across a change within one process: could not verify, each verb is one process; `TestOriginNamespaceAt` holds it. |
| 3 | The prime records branch key and the orphan sweep | FAIL | Unset, empty, padded, missing and right behave as asked, each refusal with a `git config` way forward (K); `TestCleanup_RefusesWhenPrimaryWeftBranchIsUndeterminable` reaches them. An existing branch that is not the prime's exposes `main-weft` (K, F7). A joined clone: could not verify, `clone` is refused (P0). |
| 4 | The seed's refusal wording and the seeded records branch's upstream | OK | S1, S1c, S2, P1. |
| 5 | The previous round's fixes | WARN | All five hold on a fresh hub, four with a pre-fix control (P1, S1c, S4c, Lc); F3's control is round 1's P4 row. Each fix's test fails without its fix, by reading and by the fixer's recorded sabotage. No test reaches the kept-hub path or a claim made by the push refspec alone. |
| 6 | Regions no round has driven | could not verify | Every scenario needs a refused verb (P0, Refused verbs). Driven by plain git only: the delete of a present namespaced branch (WF). |
| 7 | Safety pass | WARN | A killed lym leaves a seeded origin and an unlisted hub whose board cannot be pushed (S9); its refusal names a rerun with `--reset`, which could not be driven. No start-walk commit was left unpushed on a healthy hub: `_bolt` read `## main...origin/main` after the reader verbs on `hub-211810557`. |

What the round did not drive: every pair verb, the join, the `--single-repo` refusals, two clones, lym's own remote delete, origin variations and a rerun over a leftover.
What the method failed to show: S6 and S5 reorder time by stopping lym, so they show what lym does with the read it took, not the unassisted interleaving;
and the hub removal in S6 cannot be told apart from the sandbox's own removal of its container, so it rests on reading `pushCloneSeed`'s `fail`.

## Findings

### [BLOCKING] A records push that ends without origin's answer is rolled back blind

**Location:** `internal/fabricengine/destroy.go#pushCloneSeed`, the records push's fail path at lines 1754-1756.

**Issue:** CONFIRMED in S6.
The F1 fix asks `seedPushOutcomeUnknown` only about the board push.
A records push that ends the same way goes to `fail(..., "")`, which restores nothing and removes the hub.
In S6 the client `git push` of `refs/lym/heads/main-weft` was killed while origin's hook held it;
origin applied the ref eight seconds later, and lym's whole response was `the seed push of the records branch failed ... way forward: run the same clone again`.
Origin is left holding a records branch the precondition check did not observe, with no hub.
Nothing on origin reads as a hub, and `TestCloneSingleRepo_FailedSeedPushLeavesNothing`'s leftover rows show a rerun replaces such a branch under its lease, so the harm is a stray ref.
It is BLOCKING because it breaks `PATTERN-single-repo-hub`'s "A failed seed removes what it pushed and leaves nothing behind",
and the operator's ask to kill the push at each step and find the records branch restored.
The same sentence of the pattern is also untrue of the kept-hub path the fix added (S5, S8), which `doc.go` states and the pattern does not.
No test reaches that kept-hub path: the one new test kills the client after the board landed.
Repro: `GIT_TEMPLATE_DIR` with a `pre-receive` that sleeps 8 s on `refs/lym/heads/main-weft`, `D harden sandbox new --single-repo`,
SIGKILL the client `git push --force-with-lease=refs/lym/heads/main-weft:` during the sleep, list origin once the hook returns.

### [BLOCKING] The prime-key refusal spells the suffix literally and its hint invites the wrong branch

**Location:** `internal/fabricengine/cleanup.go:369`, the refusal the F3 fix added, and its twin at line 360.

**Issue:** CONFIRMED by dry run in K.
The new format string ends `to <prime-branch>-weft`.
`branchname.go` states that code outside `internal/weftname` never spells that token, and `PATTERN-code-conventions` binds a name that can go stale to one named constant.
The fix copied the literal from line 360 instead of moving both to the constant, so it is BLOCKING by the constraint rule; the harm of the literal alone is small.
The placeholder is also the dangerous part of the message.
The key guards the sweep exactly when the prime pair sits on another branch, and there `<prime-branch>` reads as that branch.
With the prime on `alt`, a key of `alt-weft` passes the new existence check and `cleanup --remote` lists `main-weft` with no protection.
A key of `main`, the board branch, which no records branch can be, passes too.
Fabric cannot tell a wrong suffixed branch from the right one, since the key is the only record, so the message has to carry that.
Repro: move the code store to `alt` and the records store to `alt-weft`, set `lym.primeRecordsBranch` to `alt-weft` or `main`, run `D fabric cleanup --remote`.

### [LOW] A store that keeps only its merge setting still reads as two repositories

**Location:** `internal/fabricengine/recordslayout.go#recordsStoreClaim`.

**Issue:** CONFIRMED in LK.
`recordsStoreClaim` reads `remote.origin.push` and `lym.primeRecordsBranch`.
F2 named a third mark, `branch.main.merge refs/lym/heads/main`, and the claim does not read it.
A store that lost the other two and its namespaced fetch refspec, but kept its origin, is read as two repositories:
the sweep runs unguarded, and `board sync` pushes `refs/heads/main:refs/heads/main`.
Against an origin whose code default branch is not `main`, that push answered `ok: true` and put the board at plain `refs/heads/main`.
Against one that holds `main`, the push is rejected and the way forward offers to reset or merge the board onto the code branch.
The precondition is three lost settings, so it is LOW.
An empty config is the same read with no origin to push to; no mark is left there to claim by, and I ask for no change.

## Observations below the incremental bar

These are new, not BLOCKING, and so are not findings in an incremental round.

- The charter says to run `harden sandbox` verbs with the lym on the path.
  That lym seeds in process with its own code, here the commit before the fixes, so the charter's sandbox hub is seeded by code that is not under review.
  The fixer report notes the same. The site is `internal/hardencli/cli.go`, outside the modules.
- `internal/fabricengine/pushclassified.go:200` says a board with no upstream "was never seeded onto origin".
  In S9 origin holds the whole seed, because the push outlived the killed lym.
  The named rerun then refuses with the join way forward, so the operator arrives in two hops.
- S1's message says origin took the records branch and does not say the seed already removed it again.
- The refusals of L name `git -C <hub>/_bolt config ...` from `board sync` and `git -C <store> config ...` from `cleanup`; both write the same shared config.

## Deferred item re-judged

Round 1's live gate was red with `[setup failed]` for the three packages, deferred as outside the modules.
`hardenLiveArgs` (`internal/loomcli/hardenwiring.go:242`) still appends the packages verbatim.
It stays open and outside this campaign's write surface; a tmux session named `harden-live-bare-package-paths` exists on this machine, which I did not open.

## Teardown check

```
$ lym harden sandbox clear
{"ended":[],"ok":true,"root":"/home/knatte/Code/loomyard-LYMHUB/harden-fabricengine/.lym/harden-sandbox"}
$ pgrep -af "harden-sandbox"
(seven processes of another run named harden-sandbox-fabric-verbs: lym batten, its konsole, its tmux attach)
$ pgrep -af "\.lym/harden-sandbox"
(none: no process names the sandbox root)
$ tmux ls 2>&1 | grep -i "harden-sandbox"
harden-sandbox-fabric-verbs: 1 windows (created Sun Nov 22 12:29:15 2026) (attached)
$ ls .lym/harden-sandbox
hubs.json  hubs.json.lock
$ git -C . for-each-ref refs/lym refs/tags | head
(seven refs/tags/archive-owner/... and refs/tags/archive/... tags of earlier tasks, none of this round)
$ lym harden sandbox list
{"hubs":[],"ok":true}
```

The `harden-sandbox-fabric-verbs` session and processes belong to another run of this machine, created at 12:29 while this round's last drive waited;
only their name holds the string, and none names the sandbox root.
`clear` removed the six listed hubs, S9's unlisted hub and the `r2` fixture directory.
Two drives of mine hung on a process pattern and were ended by pid, their one stray container removed by its literal path.
`git status` of the worktree is clean, and no ref of this round reached the worktree's own repository.

## Focus departures

- "Sabotage the new handling and watch the scenario fail" and "Sabotage the check by restoring the old read and watch the drive fail."
  The review half may not edit a source file.
  The control is the lym on the path, built at the commit before the fixes: it seeded S1c and S4c and ran the verbs of Lc on its own hub.
  That departs from the rubric's "invoke only the `.dev-bin` lym against a sandbox hub"; the path lym is used there as the wrong binary on purpose.
- "Use the `.dev-bin` lym for hub verbs, and the lym on the path for `harden sandbox` verbs."
  The seeds under review ran as `.dev-bin/lym harden sandbox new`, because that verb seeds in process and the path lym predates the fixes.
  `clear` and `list` ran with the path lym.
- "Keep the sandbox origin alive past the hook's return."
  The sandbox removes its container when the seed fails, so origin was kept alive by stopping lym, as Method describes.
- "Repeat on a joined clone if a join can be made; otherwise record that it could not be verified, with the refusal text."
  It could not be verified; the refusal text is P0's.
- "Regions no round has driven on the substrate."
  The archive tags, the join and its config, the `--single-repo` refusals, two clones, lym's remote delete, a pair verb killed mid-operation and the origin variations
  each need `clone`, `add`, `remove`, `push` or `cleanup --apply`, all refused (P0).
  No concurrency claim is made, so none needs a sequential control or a second hub.
- "the answer is not cached across a change within one process."
  A CLI drive is one process per verb, so this rests on `TestOriginNamespaceAt`'s cache rows, which I read and did not run.
- "Log as you go."
  The log lived in a scratch file until the end, for the reason Method gives.
