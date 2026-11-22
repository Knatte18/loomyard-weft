---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/fabricengine/destroy.go#pushCloneSeed
    summary: "A seed whose board push fails without a definite rejection (client killed, connection dropped, a timeout) deletes the records branch and removes the hub, while origin's receive-pack can still apply the board afterwards, leaving origin holding refs/lym/heads/main, which reads as a hub, with no records branch and no hub; PLAUSIBLE, from P9's live sequence and its plain-git control."
    fix: "In pushCloneSeed's fail path (destroy.go:1720), when the board push returns anything but a definite rejection (the GitError carries no `[remote rejected]` or `! [rejected]` line for refs/lym/heads/main, or exit code -1), first read origin's board with `git ls-remote origin refs/lym/heads/main` and delete a board found at boardTip with DeleteRemoteBranchLeased(origin, OneRepositoryBoardBranch, boardTip, ns). Restore the records branch only after that. When the read fails, or the board is not yet visible, keep the hub and return an error naming both refs and the commands to check and delete them by hand, instead of removing the hub. Add an integration test whose bare's pre-receive hook sleeps on the board ref while the test kills the client push, then asserts origin holds no refs/lym/heads/main."
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/fabricengine/recordslayout.go#originNamespaceAt
    summary: "A records store whose remote.origin.fetch is removed or set to +refs/heads/* is read as two repositories, although its push refspecs, branch.main.merge refs/lym/heads/main and lym.primeRecordsBranch still name the namespace; the board then pushes to origin's plain refs/heads/main, the code branch (rejected only for unrelated history), and the sweep skips the prime-key check; a store with no origin remote or an empty config also reads as identity, not as an error; CONFIRMED in P5 and P12."
    fix: "In originNamespaceAt, after reading the fetch refspecs, also read remote.origin.push and the PrimeRecordsBranchKey of the store (one `git config --get-regexp` or the tolerant config read). When either names RecordsNamespaceRoot but no fetch refspec maps RecordsNamespaceRoot+RecordsNamespaceHeads, return an uncached error naming the store and the way forward: restore `remote.origin.fetch` with `git -C <store> config --replace-all remote.origin.fetch '+refs/lym/heads/*:refs/remotes/origin/*'` and `--add ... '+refs/lym/tags/*:refs/tags/*'`. Answer a missing origin remote with an error too when the store holds PrimeRecordsBranchKey. Add rows to TestOriginNamespaceAt for an unset fetch refspec, a +refs/heads/* refspec and a removed origin, each with the key set."
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/fabricengine/cleanup.go:356
    summary: "primaryWeftBranch accepts a lym.primeRecordsBranch naming a branch that does not exist, so once the prime pair is on another branch the orphan sweep marks the real prime records branch main-weft deletable (protected absent), which --apply --remote would delete locally and on origin; CONFIRMED by dry run in P4."
    fix: "In primaryWeftBranch's one-repository branch (cleanup.go:352-362), after reading the key, check that refs/heads/<key> exists in the records store (gitrepo.New(recordsDir).BranchExists). When it does not, return the fail-closed error naming the key's value, that no such branch exists, and the same `git -C <store> config lym.primeRecordsBranch <prime-branch>-weft` way forward. Add a row to TestCleanup_RefusesWhenPrimaryWeftBranchIsUndeterminable for a key naming a missing branch, run with the prime pair checked out on another branch."
  - id: F4
    severity: LOW
    class: consistency
    location: internal/fabricengine/destroy.go#seedRefusedByOrigin
    summary: "A pre-receive hook that declines the board push is reported as \"origin refused the seed push ... use a host that accepts refs outside refs/heads/ and refs/tags/\", although the same origin accepted refs/lym/heads/main-weft moments before; the way forward misnames the cause; CONFIRMED in P7."
    fix: "Give seedPushCause the knowledge that the records push already landed (pass recordsLanded bool from pushCloneSeed, true for the board push). When it did, a `[remote rejected]` of the board is a refusal of that ref by the origin's own rules, so word the way forward as: origin's hooks or rules declined refs/lym/heads/main, the records branch was restored, fix or lift the rule and run the same clone again. Keep the host-refusal wording for a records push the origin rejected."
  - id: F5
    severity: LOW
    class: scope
    location: internal/fabricengine/destroy.go:1767
    summary: "The seed pushes the prime's records branch to refs/lym/heads/main-weft but sets an upstream only for the board, so git's @{u} on the clone-made main-weft answers no upstream configured and git status shows no ahead count, while the sandbox's own commit already sits one past origin; CONFIRMED in P1."
    fix: "In pushCloneSeed, add branch.<seed.recordsBranch>.remote origin and branch.<seed.recordsBranch>.merge ns.Branch(seed.recordsBranch) to the upstream settings written after the board lands, and assert both in TestCloneSingleRepo_SeedsTheNamespace with `git rev-parse @{u}` on the records branch."
---

The review is below the probe log: the focus walk, then one block per finding.

## Probe log

Variables used below:
`L` is `/home/knatte/Code/loomyard-LYMHUB/harden-fabricengine/.dev-bin/lym`, the `.dev-bin` lym `lym harden sandbox new` printed;
`S1` is `.lym/harden-sandbox/hub-212252504`, the first one-repository sandbox;
`H1` is `$S1/origin-bare-LYMHUB` and `O1` is `$S1/origin-bare`, its origin.

### P1 — build the one-repository sandbox

Command: `lym harden sandbox new --single-repo` (11 s).
Printed `{"hub":".../hub-212252504/origin-bare-LYMHUB","lym":".../.dev-bin/lym","ok":true,"single_repo":true}`.

Observed:

- `git ls-remote $O1`: `HEAD` and `refs/heads/main` at `fafe5a6`, `refs/lym/heads/main` at `2cd5bee`, `refs/lym/heads/main-weft` at `60e7fa5`.
  No ref under plain `refs/heads/` or `refs/tags/` besides the code's `main`.
- Records store `origin-bare-weft` config: fetch `+refs/lym/heads/*:refs/remotes/origin/*` and `+refs/lym/tags/*:refs/tags/*`,
  push `refs/heads/*:refs/lym/heads/*` and `refs/tags/*:refs/lym/tags/*`, `tagopt --no-tags`, `lym.primerecordsbranch main-weft`,
  `branch.main.merge refs/lym/heads/main`. Matches the charter's refspec list.
- Code store `origin-bare`: default refspec, `branch.main.merge refs/heads/main`, refs `main`, `origin/main`, `origin/HEAD` only.
- `_bolt`: `git status -sb` reads `## main...origin/main`, and `git rev-parse @{u}` answers `2cd5bee`: the board's namespaced upstream works.
- Records store on `main-weft`: `git rev-parse @{u}` answers `fatal: no upstream configured for branch 'main-weft'`,
  and `git branch -vv` shows no upstream on `main-weft`, although the seed pushed it to `refs/lym/heads/main-weft`.
  The local tip `b1ac214` (the sandbox's own Tier 2 commit) is one commit past origin's `60e7fa5`.

### P2 — which verbs a task session may run

Command: from `$H1/origin-bare`, `$L fabric <verb>` for each verb of the charter.

Refused, each with the same text
`lym fabric <verb> is refused from the task session ly:harden-fabricengine:harden-burler-review; build a scratch hub as a hubforge test fixture instead, and when the work itself needs this verb, stop and report status: FAILED so the run ends stuck and the orch runs it`:
`add`, `push`, `pull`, `sync`, `reconcile`, `checkout`, `remove`, `clone`.

Allowed: `status`, `pairs`, `list`, `cleanup` and `prune` as dry runs, `shortname`, and `lym board list`.
`cleanup --apply --remote` is refused with the same text, so `cleanup` and `prune` run only as dry runs.
Every scenario that needs a refused verb is reported below as could not verify, with this refusal as the reason.

### P3 — orphan records branch beside a same-named code branch (dry run)

Fixture, by raw git: in the records store `git branch t1-weft main-weft`, pushed to `refs/lym/heads/t1-weft`;
from the code store `git push origin main:refs/heads/t1-weft`, a code branch of the same name.

`$L fabric cleanup --remote` (dry run) answered `entries` `main` protected and `t1-weft` not protected, `ok: true`,
and `warp_skipped_reason` `task branches on origin were kept: the open pull requests could not be listed: githubclient: parse owner/repo from "<O1>": unrecognized remote URL shape`.
The warp-origin sweep never runs on a local bare origin, so the remote cleanup of code task branches could not be verified.
`$L fabric cleanup --apply --remote` was refused (P2), so the delete itself, its mutation record and its idempotence could not be verified.
`git ls-remote $O1` before and after: unchanged.

### P4 — the prime records key (dry run)

Each row edits `lym.primeRecordsBranch` in the records store, then runs `$L fabric cleanup --remote`.

| Key | Prime pair on | Result |
|---|---|---|
| unset (`git config --unset`) | `main`/`main-weft` | refused: `the records store at <R> has no lym.primeRecordsBranch; refusing to enumerate orphan weft branches — set it with \`git -C <R> config lym.primeRecordsBranch\` to <prime-branch>-weft` |
| empty | `main`/`main-weft` | the same refusal |
| `nosuch-weft` | `main`/`main-weft` | `ok: true`; `main-weft` not listed (its code sibling `main` is checked out) |
| `t1-weft` | `main`/`main-weft` | `ok: true`; `t1-weft` now protected |
| `main-weft` | `other`/`other-weft` (raw `git checkout -b`, standing in for the refused `lym fabric checkout`) | `main-weft` protected: true |
| `nosuch-weft` | `other`/`other-weft` | `main-weft` listed with no `protected`, which the help defines as "`--apply` would delete this" |
| ` main-weft ` (spaces) | `other`/`other-weft` | `main-weft` protected |

The key is the only protection of the prime records branch once the prime pair is on another branch:
the correct key protects it, and a key naming a branch that does not exist lets the sweep take it (F3).

### P5 — layout detection under a faulted records config (dry run)

Each row edits the records store's config (restored from a copy between rows), then runs `$L fabric cleanup --remote`.

| Fault | Key | Result |
|---|---|---|
| none | unset | refused, no `lym.primeRecordsBranch` (the one-repository path) |
| `remote.origin.fetch` unset, push refspecs and key kept | set | `ok: true` |
| `remote.origin.fetch` unset | unset | `ok: true`, `t1-weft` deletable: the key check was skipped, so the store was read as two repositories |
| `remote.origin.fetch` `+refs/heads/*:refs/remotes/origin/*` | unset | `ok: true`, read as two repositories |
| `remote.origin.fetch` `not a refspec` | set | error `cannot read the records store at <R> (fabricengine: read the layout of the repository at <R>: gitrepo: open go-git handle at ...` |
| `origin` remote removed | unset | `ok: true`, `remote_skipped_reason` `no remote deletion attempted: the weft repo has no "origin" remote configured` |
| config truncated to empty | set | `ok: true`, the same `remote_skipped_reason` |
| config holding an unterminated `[remote "origin` section | set | the go-git open error above |

The sabotage proof is the first two key-unset rows: the healthy store takes the one-repository path and refuses,
the store with its fetch refspec removed takes the two-repository path and proceeds,
although its push refspecs, `branch.main.merge refs/lym/heads/main` and `lym.primeRecordsBranch` all still name the namespace (F2).
`lym fabric status` on the faulted store answered `{"changes":[],"failed_pushes":[],"merge_in_progress":false,"ok":true}`.

### P6 — fault injection into the seed: method

`lym fabric clone` is refused (P2), so the seed is driven through `lym harden sandbox new --single-repo`,
which runs `fabriccli.CloneAndWireFromLocalOrigin` in process, from the path lym built at this worktree's HEAD `2dbfdc314`.
A first try put a `pre-receive` hook in place through `GIT_CONFIG_COUNT`/`core.hooksPath`; it never fired,
because git drops `GIT_CONFIG_*` for the receive-pack of a local push (checked with a plain `git push`).
The method used: a background loop polls the sandbox root every 5 ms for the new `hub-*/origin-bare/hooks` and copies a scenario's
`pre-receive` (logs each ref, rejects or sleeps on one named ref) and a `post-receive` (logs `git for-each-ref` of origin) into it,
before the clone reaches its seed push.
Each plant was confirmed by the hook's own log line.

### P7 — origin rejects the board ref

Hook rejects `refs/lym/heads/main`. `lym harden sandbox new --single-repo` answered:
`origin refused the seed push of the board: gitrepo: git push --force-with-lease (leased at ): ... ! [remote rejected] 651b30d -> refs/lym/heads/main (pre-receive hook declined) ...; way forward: use a host that accepts refs outside refs/heads/ and refs/tags/, or make a two-repository hub with \`lym fabric clone <records-url> <code-url>\``.

Hook log, in order: `refs/lym/heads/main-weft 0000000 -> b912821` (accepted), `refs/lym/heads/main 0000000 -> 651b30d` (rejected),
`refs/lym/heads/main-weft b912821 -> 0000000` (the restore, a leased delete).
So the records branch goes first, the board last, and the rollback restores the records branch to its observed absence.
The sandbox removed its `hub-*` directory, origin included, so the hub's removal by fabric itself and origin's end state could not be listed separately.
The way forward claims the host refuses refs outside `refs/heads/`, although the same origin had accepted `refs/lym/heads/main-weft` a moment before (F4).

### P8 — origin rejects the records ref

Hook rejects `refs/lym/heads/main-weft`. The clone answered `origin refused the seed push of the records branch: ... (pre-receive hook declined) ...` with the same way forward.
Hook log: one line, the rejected `refs/lym/heads/main-weft`; no board push followed and `post-receive` never ran, so origin took nothing.

### P9 — the board push dies mid-flight

Hook sleeps 20 s on `refs/lym/heads/main`; a background `pkill -KILL -f` aimed at the client `git push --force-with-lease=refs/lym/heads/main:` fired one second into the sleep.
The pattern also matched the probe's own shell, so that run's printed output was lost; the hook logs survived:

- `refs/lym/heads/main-weft 0000000 -> 39fda24` accepted, `post-receive` shows origin with `refs/heads/main` and `refs/lym/heads/main-weft`.
- `refs/lym/heads/main 0000000 -> ccf04de` entered the hook and was still sleeping when,
  11 s later, `refs/lym/heads/main-weft 39fda24 -> 0000000` arrived: the clone had read the killed push as failed, restored the records branch and torn down.

Control, plain git under the sandbox root: a bare whose `pre-receive` sleeps 5 s, a client `git push HEAD:refs/lym/heads/main` sent SIGKILL after 1 s, and the same with SIGTERM.
`git ls-remote` 7 s later lists both refs: receive-pack finishes the update after its client is gone.
Put together, a seed whose board push dies after origin received it deletes the records branch and removes the hub, and then origin takes the board:
origin is left holding `refs/lym/heads/main`, which a one-URL clone reads as a hub, with no records branch and no hub (F1).
The last step could not be watched on the sandbox hub, because the sandbox removed the origin before the hook returned.

### P10 — the seeding lym process is killed during the board push

Hook sleeps 5 s on `refs/lym/heads/main`; `pkill -KILL -f '^lym harden sandbox new'` fired 0.5 s into the sleep (exit 137).
`post-receive` 5 s later: origin holds `refs/heads/main`, `refs/lym/heads/main` `f8a3ab9` and `refs/lym/heads/main-weft` `63dd608`:
the `git push` child outlived the killed lym and the seed landed whole.
The hub directory `hub-3728750445/origin-bare-LYMHUB` stays, and `lym harden sandbox list` does not list it.
Its records store has no `branch.main.remote`/`branch.main.merge` (set only after a seed push returns), so `_bolt` reads `## main` with no upstream.
`$L fabric status`, `$L board list` and `$L fabric cleanup` in it answer `ok: true`; the start walk of those verbs committed two board commits (`lym: seed stencils`, `lym: seed specs`) that nothing pushes and no verb reports.

### P11 — the seeding lym process is killed before the seed push

Two runs, each sending SIGKILL to `lym harden sandbox new --single-repo` when a marker file of the new hub appears.

- On the records store's `.git`: the kill landed during the board worktree's creation.
  The hub holds `origin-bare` and `origin-bare-weft`, no `_bolt`; the records store is fully configured (refspecs, `tagopt`, the prime key) with a local `main-weft`.
  `git ls-remote` of origin: `HEAD` and `refs/heads/main` only, nothing under `refs/lym/`.
  `$L fabric status` and `$L fabric cleanup` answer `not initialized here; run "lym fabric reconcile"; ...`, and `$L board list` `board config .../board.yaml not initialized; run "lym fabric reconcile"; ...`.
  Running that `reconcile` is refused (P2), so whether it would push the board unleased or out of order could not be verified.
- On `_bolt`'s first commit (`.git/worktrees/_bolt/logs/HEAD`): origin holds `refs/lym/heads/main-weft` and no board,
  so the kill fell between the two seed pushes; nothing on origin reads as a hub.
  The records store has no board upstream; `_bolt` holds the local commit `fabric clone: record repo-wide config`.

A rerun of `lym fabric clone --single-repo` in the same container, with and without `--reset`, is refused (P2): could not verify.

### P12 — board writes reach the namespace, and leave it when the fetch refspec is gone

In `H1`, `$L board upsert '{"slug":"probe-a","title":"probe a","labels":["bug"]}'`: `ok: true`; origin's `refs/lym/heads/main` moved `2cd5bee -> 751eaac`, `refs/heads/main` stayed `fafe5a6`.

Sabotage: `git config --unset-all remote.origin.fetch` in the records store, push refspecs, `branch.main.merge refs/lym/heads/main` and the prime key kept, then a second upsert `probe-b`.
The verb answered `ok: true` and committed `0f2d06f` locally; origin did not move.
The `_bolt` trace log `_bolt/.lym/logs/trace-...-1965681.log` holds three
`fabricengine: rejected push retried ... side=bolt branch=main ... error="gitrepo: push rejected: hint: Updates were rejected because the remote contains work that you do not ... gitrepo: resolve upstream refs/remotes/origin/lym/heads/main: reference not found"`
and `a push of bolt branch main failed (other)`:
the board's push went to origin's plain `refs/heads/main`, the code branch, and only its unrelated history made git reject it (F2).
`$L fabric status` afterwards answered `failed_pushes: []`.
With the config restored, `$L board sync` answered `ok: true` and origin's `refs/lym/heads/main` moved to `0f2d06f`.

### P13 — the archive tag of a discarded task branch

`lym fabric remove` is refused (P2), so no task branch could be discarded on the sandbox hub; this is settled from the code only and stays unverified on the substrate.
`DiscardTaskBranch` (`internal/fabricengine/taskbranchdiscard.go:42`) archives from `repoDir := l.WorktreePath()`, the code store,
and `pushArchiveTag` (`archive.go:224`) maps the tag through `originNamespaceAt(repoDir)`, the identity namespace of a code store:
the code store pushes the task branch's archive as `refs/tags/archive/<slug>/<sha12>`, plain `refs/tags/` on the shared origin.
The records branch's archive goes through `tagAndPushArchiveTip(rec, weftRoot, ...)` (`archive.go:168`) from the records store, whose namespace maps it to `refs/lym/tags/archive/<slug>/<sha12>`.
The sighting is consistent with that: the plain tag is the code store's archive of a code commit, a code ref, not a records ref that leaked.
Trial 2's abandon pushed no code archive (its `archive_tags` was null) only because its code branch sat on `main`'s tip.

### P14 — teardown check

```
$ lym harden sandbox clear
{"ended":[],"ok":true,"root":"/home/knatte/Code/loomyard-LYMHUB/harden-fabricengine/.lym/harden-sandbox"}
$ pgrep -af "harden-sandbox" || echo ...
no process names the sandbox root
$ tmux ls 2>&1 | grep -i "harden-sandbox" || echo ...
no sandbox tmux session
$ ls .lym/harden-sandbox
hubs.json  hubs.json.lock
$ git -C . for-each-ref refs/lym refs/tags | head
(seven refs/tags/archive-owner/... and refs/tags/archive/... tags of earlier tasks of this repository, none named after a probe of this round)
$ lym harden sandbox list
{"hubs":[],"ok":true}
```

`clear` also removed the four hub directories the killed seeds left unlisted (P10, P11), the hook directory and the plain-git control.
No ref of this round was pushed to the worktree's own repository.

## Focus walk

| # | Entry | Status | Evidence |
|---|---|---|---|
| 1 | Seed push order and rollback | FAIL | Order and both rejection rollbacks hold (P7, P8); a board push killed mid-flight can land after the rollback (P9, F1); the way forward misnames a hook decline (F4). Empty origin, a pre-existing `main-weft`, a rerun and the lease against a second clone: could not verify, `clone` refused (P2). |
| 2 | Failure between hub creation and the seed push | WARN | A killed lym leaves its hub on disk and nothing on origin, and the verbs name `reconcile` (P11); a kill during the board push leaves a seeded origin and a hub with no board upstream (P10). Whether `reconcile` or a rerun pushes unleased: could not verify, both refused. |
| 3 | Joining | could not verify | `clone` refused (P2). |
| 4 | Refusals | could not verify | `clone` refused (P2); `sandbox new` takes no origin of the caller's. |
| 5 | Namespace leaks on origin | WARN | A board write lands under `refs/lym/heads/main` and leaves the code `main` alone (P12); the archive tag of a discarded task branch is the code store's, under plain `refs/tags/archive/<slug>/<sha12>`, settled from the code only (P13). Every pair verb: could not verify, `add`, `push`, `remove` and `cleanup --apply` refused. |
| 6 | Two clones of one hub | could not verify | A second clone needs `clone`, refused (P2); no concurrency claim is made, so none needs a second-hub proof. |
| 7 | Remote delete | could not verify | `remove` and `cleanup --apply --remote` refused (P2, P3). |
| 8 | Prime records branch | FAIL | Written by the clone (P1); unset and empty fail closed with a `git config` way forward (P4); a key naming a missing branch exposes `main-weft` to the sweep (P4, F3). A joined clone: could not verify. |
| 9 | Layout detection | FAIL | A removed or identity fetch refspec reads as two repositories and the board push goes to plain `refs/heads/main` (P5, P12, F2); a bad refspec value and a corrupt config are errors (P5). |
| 10 | Config read and upstream | WARN | The board's namespaced `@{u}` works (P1); the seeded `main-weft` has none (F5). Pair branches, odd names and a bad `branch.<b>.merge`: could not verify, no pair can be made. |
| 11 | Crash and rebirth of a one-repository pair | could not verify | Every pair verb is refused (P2). |
| 12 | Environment and origin variations | could not verify | `clone` refused; `sandbox new` makes only a local-path origin whose HEAD is `main`. |

What the round did not drive: every pair verb, the join, every refusal, two clones, the remote delete and the warp-origin sweep (P3: it needs a GitHub URL).
What the method failed to show: P9's last step, the board landing after the rollback, could not be watched on the sandbox, because the sandbox removed its origin before the hook returned.
It rests on the plain-git control.

## Findings

### [BLOCKING] A board push that dies mid-flight can land after the seed's rollback

**Location:** `internal/fabricengine/destroy.go#pushCloneSeed` (fail path at line 1720, board push at 1762).

**Issue:** PLAUSIBLE.
`pushCloneSeed` treats every board push error as "the board did not land": it restores the records branch, which deletes `refs/lym/heads/main-weft` when origin held none, and removes the hub.
An error that is not a rejection proves nothing about origin, and P9's control shows a receive-pack applies the update after its client is gone.
In P9 the clone deleted `main-weft` 11 s after the board push entered origin's hook, with that hook still running.
The result is an origin holding a board and no records branch, and no hub to fix it from.
A later `lym fabric clone --single-repo` of it then refuses with "already holds a one-repository hub's board ... join it", and a join gets a hub whose prime records branch is missing.
This breaks `PATTERN-single-repo-hub`'s "A failed seed removes what it pushed and leaves nothing behind", which this round binds as BLOCKING.
Repro: plant a `pre-receive` in the seed's origin that sleeps 20 s on `refs/lym/heads/main`, kill the client `git push --force-with-lease=refs/lym/heads/main:` one second in, then list origin once the hook returns.

### [MEDIUM] A faulted fetch refspec reads as two repositories, and the board pushes to the code branch

**Location:** `internal/fabricengine/recordslayout.go#originNamespaceAt`.

**Issue:** CONFIRMED.
With `remote.origin.fetch` removed from the records store, every other key still naming `refs/lym/`, a board upsert pushed to origin's plain `refs/heads/main`, the code branch.
Only its unrelated history made git reject the push, and the verb answered `ok: true` while `fabric status` reported no failed push (P12).
The same fault, or a `+refs/heads/*` refspec, makes the orphan sweep take the two-repository path and skip the prime-key check (P5).
For a records branch with no namespace twin on origin, the same push creates `refs/heads/<slug>-weft` on the shared origin, a namespace leak, and `cleanup --remote` would delete under `refs/heads/`.
That last step was not driven, because `push` and `cleanup --apply` are refused.
A store with no origin remote or an empty config also answers identity, where the charter asks for an error.

### [MEDIUM] A prime key naming a missing branch lets the sweep delete the prime records branch

**Location:** `internal/fabricengine/cleanup.go:356`.

**Issue:** CONFIRMED by dry run, whose verdicts the help says match `--apply`.
With the code prime on `other` and the records prime on `other-weft`, as after a `lym fabric checkout`, the key `main-weft` keeps `main-weft` protected.
The key `nosuch-weft` lists `main-weft` with no protection, so `cleanup --apply --remote` would delete the prime's durable records line locally and on origin (P4).
Unset and empty keys already fail closed; a key that names nothing is the same fault and passes.

### [LOW] The seed's way forward blames the host for a ref its own hook declined

**Location:** `internal/fabricengine/destroy.go#seedRefusedByOrigin`.

**Issue:** CONFIRMED.
When origin's `pre-receive` declined only `refs/lym/heads/main`, the clone said to use a host that accepts refs outside `refs/heads/` and `refs/tags/` (P7).
That origin had just accepted `refs/lym/heads/main-weft`, so the advice sends the operator to a two-repository hub for a fault that a changed hook would clear.

### [LOW] The seeded prime records branch gets no upstream

**Location:** `internal/fabricengine/destroy.go:1767`.

**Issue:** CONFIRMED.
After a clean seed, `git rev-parse @{u}` on `main-weft` answers `no upstream configured`, and `git status -sb` reads `## main-weft` with no ahead count.
The branch sits one commit past origin (P1): the seed pushed it but recorded neither `branch.main-weft.remote` nor `.merge`.
The charter asks that `@{u}` work on namespaced records branches the clone made.

## Focus departures

- "Settle the archive-tag question first: on a fresh `--single-repo` hub, discard a task branch (create a pair, push, remove it so its task branch is archived), and list where the archive tag ... landed on origin."
  `add`, `push` and `remove` are refused from this task session (P2), so the question was settled from the code (P13) and stays undriven on the substrate.
- "Prove each concurrency claim with a sequential control and a second-hub run."
  No second clone of one origin could be made, since `clone` is refused, so no concurrency scenario was driven and the round makes no concurrency claim.
- "a sabotage proof for each green drive of a mechanism (the namespace parameter, the lease, the order of the seed, the existence check before a delete)."
  The namespace parameter (P12), the prime key (P4), the layout read (P5) and the seed order (P7) have one;
  the lease and the existence check before a delete sit behind refused verbs and were not driven.
- Every focus entry whose scenario needs `clone`, `add`, `push`, `pull`, `sync`, `reconcile`, `checkout`, `remove` or `cleanup --apply` is reported as could not verify.
  The charter names that outcome for a verb refused to a task session; the orch has to run those scenarios.
