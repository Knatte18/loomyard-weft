---
verdict: APPROVED
---

Round 4 drove every focus entry live on one-repository sandbox hubs, with a two-repository hub as the control for focus 6.
The frontmatter holds no finding because the incremental rule admits none outside the changed `gatecli`, `gateslot` and `loomcli` hunks.
Five defects in the campaign's modules are recorded below the frontmatter instead, one of them MEDIUM.
Round 3 raised no finding, so no `prior` entry is owed.

## Focus walk

| Focus | Verdict | Probe |
|---|---|---|
| 1, killed discard and its re-run | OK; the way forward names no command (O5) | P1 |
| 2, keep-local tag pushed at `--remote` | WARN; a kept local records tag is pushed with no output or help naming it (O3) | P3 |
| 3, failed step without a kill | OK | P2 |
| 4, `adopt --archives` over owner tags | WARN; read side OK in two hubs, push path blocked as named | P4 |
| 5, plain `refs/tags/archive/` on origin | WARN; no verb names the ref, no teardown loses the commit (O4) | P5 |
| 6, relative URL, two-repository control | OK | P6 |
| 7, layout read before `Fetch`/`Pull`/upstream reads | WARN; 55 calls tabled, `lym fabric merge` reads none (O2) | P7 |
| 8, kills of checkout, pull, reconcile, clone | FAIL for a killed clone (O1); pull, reconcile OK; checkout has no kill window; hand-over paths blocked as named | P8 |
| 9, sabotage proofs and concurrency twice | OK for the split, the record, the layout read and the seed; no lease lost, so a losing lease is undriven | P9 |

Earlier fixes re-driven: round 2's G1 (P6, a relative URL), G2 (P7), G3 (P1 for the discard half, which round 3 did not drive, and P2), G4 (P4).
Round 3's `--no-carry` fix: with the worktree's `.dev-bin/lym`, `lym gate test ./internal/gitrepo` run twice carried the second run and started 0 test binaries;
`lym gate test --no-carry ./internal/gitrepo` then started 1. The `lym` on the PATH answers `unknown flag: --no-carry`, so a live gate only starts its tests when the binary that runs it carries this branch.

## Observations outside the incremental rule

This round is incremental, and its changes file lists only `internal/gatecli`, `internal/gateslot` and `internal/loomcli` files.
The format admits a new finding only at BLOCKING on a changed hunk of a listed file, so none of the defects below can stand in the frontmatter.
The rubric says to record every finding, so each is recorded here, with the fields a finding carries.
The first is MEDIUM: under the rubric's convergence rule it should keep the campaign from converging, which the frontmatter cannot say.

- **O1 — MEDIUM, design, CONFIRMED.**
  Location: `internal/fabriccli/fabric.go:673` (the reconcile verb's `NewBolt(boltDir).PullThenCommitWritten`), with `internal/fabricengine/clone.go:225` and `:264` ("hub already exists").
  Scenario (P8, twice): `lym fabric clone <url> --into <dir>` killed once its records store exists and before `_bolt` does.
  `fabric status` and `fabric pull` then print `not initialized here; run "lym fabric reconcile"`;
  typed, `lym fabric reconcile` fails with the raw `gitrepo: git dir of …/_bolt: dotgit: not a git repository …`;
  a plain re-clone answers `hub already exists at …` with no way forward. Only `lym fabric clone <url> --into <dir> --reset`, which no output names, rebuilds the hub.
  Fix: in the reconcile verb, before the Bolt pull, refuse a hub whose `BoltDir` holds no repository with a way forward naming `lym fabric clone <url> --into <parent of the hub> --reset`.
  Make `CloneHub`'s "hub already exists at %s" name the same `--reset` re-run.
  Add a regression test that removes `_bolt` from a hubforge one-repository hub and asserts both messages.
- **O2 — LOW, scope, CONFIRMED.**
  Location: `internal/fabricengine/mergeguards.go:57`.
  Scenario (P7): records store's `remote.origin.fetch` unset; `lym fabric merge y1` stages the merge with `ok: true` and names no layout gap,
  so the weft SHA in the merge record comes from stale tracking refs. Every other verb driven names fabric's refspec way forward.
  Fix: call `originNamespaceAt(f.records path)` before `f.records.Fetch()` in `resolveMergeSources` and log its error with the weft fetch's warning, or return it as the verb's refusal.
  The same gap stands on non-fabric paths: `pull.go:186` and `recordsgit.go:346` via `FastForwardPair`, `codetipsettle.go:21,43` via `DobbyRuns`, `bolt.go:162` via `lym machine name`.
- **O3 — LOW, design, CONFIRMED.**
  Location: `internal/fabricengine/archive.go:294` (`pushLocalArchiveTags`).
  Scenario (P3): a records commit kept under a local `archive/t4/<sha>` tag, as `DropLocal` keeps one with "the tags stay local", is pushed to origin by a later `lym fabric remove t4 --remote`.
  No verb output names the push (the job is handed off), and `lym fabric remove --help` says nothing of it.
  Fix: decide whether `DropLocal`'s tags may leave the machine.
  If yes, name in `lym fabric remove --help` and `cleanup --help` that `--remote` pushes the slug's local archive tags origin lacks, and correct `DropLocal`'s doc.
  If no, give `DropLocal`'s tags a prefix `pushLocalArchiveTags` does not list.
- **O4 — LOW, scope, CONFIRMED.**
  Location: `internal/fabricengine/cleanupremotewarp.go:302` (`remoteTagRefs`, the records-side archive listing `cleanup` reads).
  Scenario (P5): an earlier binary's plain `refs/tags/archive/t5/<sha>` on a one-repository origin is named by no `status`, `cleanup`, `prune` or `reconcile` output, before or after the branch is discarded.
  No commit is lost: the discard pushes its own namespaced tag first.
  Fix: in `cleanup`'s read of a one-repository origin, list plain `refs/tags/archive/*` and `refs/tags/archive-owner/*` refs and report each with a way forward that pushes it under `refs/lym/code-tags/` and deletes the plain ref.
- **O5 — NIT, consistency, CONFIRMED.**
  Location: `internal/fabricengine/remoteteardown.go:187` (`discardRerunWayForward`).
  Scenario (P1): a killed discard's record names "run the command that discarded the task branch again", no command an operator can type; the re-run that works is `lym batten cancel <slug> --remove`.
  Fix: carry the discarding command in the job and the pending record, as the removal and cleanup kinds name theirs, and print it.

## Probe log

Sandbox: `lym harden sandbox new --single-repo` built hub `hub-357845542` (origin `$O`, prime `$P`, records store `$W`, Bolt `$B`), lym `.dev-bin/lym`.

**P1 — focus 1, discard killed between local tag and origin push**

- `lym fabric add t1`; commit `f1.txt` (tip `4a97394c50b7`); `lym fabric push` → origin `refs/heads/t1 4a97394`, `refs/lym/heads/t1-weft 3669716`.
- Origin `pre-receive` hook logs every ref and, for a ref under `refs/lym/code-tags/archive/t1/`, sleeps 40 s and declines.
- `lym batten cancel t1 --remove` from the prime: `pair_removed: true`, `archive_tags: null`.
  Hook log: `refs/lym/tags/archive/t1/3669716b9b1e` created and `refs/lym/heads/t1-weft` deleted in process, then `refs/lym/code-tags/archive/t1/4a97394c50b7` held.
  `git push origin refs/tags/archive/t1/4a97394c50b7:refs/lym/code-tags/archive/t1/4a97394c50b7` seen running under the `lym fabric remote-teardown --slug t1` child.
- `kill -9` the child. Local tag `archive/t1/4a97394c50b7` sits in the code store; origin still holds `refs/heads/t1 4a97394`, no code tag.
  `_bolt/.lym/teardown/t1.pending.json` kept, kind `discard`, no `task_tip` (the pair removal had already deleted the local branch, so the job read origin's tip).
- `lym fabric status` and `lym fabric prune` both list `pending_teardowns` t1 with `way_forward: "run the command that discarded the task branch again"`.
  No later verb was blocked by the dead child's lock file.
- Sabotage control: hook switched to decline without sleep; re-ran `lym batten cancel t1 --remove`.
  Origin kept `refs/heads/t1`, no code tag; the record was kept with the same way forward. So the drive enters the tag-before-delete order.
- Re-run with the hook lifted: hook log shows `refs/lym/code-tags/archive/t1/4a97394c50b7` created at second 1795419258, then `refs/heads/t1` deleted at 1795419259.
  `git -C $O cat-file -t 4a97394c50b7` → `commit`; the pending record was removed; no `refs/tags/archive/` ref on origin.
- Verdict: OK. WARN: the way forward names no exact command, since the record holds kind `discard` only; `lym batten cancel t1 --remove` re-run works.

**P2 — focus 3, a failed step without a kill**

- Pair t2 pushed (`refs/heads/t2 ef2bc44`, `refs/lym/heads/t2-weft 1f51557`); hook declines `refs/lym/tags/archive/t2/` at once.
- `lym fabric remove t2 --remote`: `ok: true`, `remote_handed_off: true`. One push attempt logged, declined; origin keeps `refs/lym/heads/t2-weft`.
  `t2.pending.json` kept with `weft_tip 1f515577…` and way forward
  `push the archive tag with \`git -C <records store> push origin refs/tags/archive/t2/1f515577c591:refs/lym/tags/archive/t2/1f515577c591\`, then when origin still holds a branch of the pair, run \`lym fabric remove t2 --remote\` again`.
- Readers: `status` and `prune` list the record with that way forward.
  `cleanup` and `cleanup --remote` (dry run) list only `main`, and `reconcile` reports the live pairs healthy; neither names t2.
  The `PendingTeardowns` doc promises the listing to `status` and `prune` only, so their silence is no departure.
- Typed the hand push with the hook in place: `! [remote rejected] … (pre-receive hook declined)`.
  Typed `lym fabric remove t2 --remote` with the hook in place: `finished: true`, origin unchanged, the record kept, now with way forward `run \`lym fabric remove t2 --remote\` again` (no local tip left to name).
- Hook lifted: the typed hand push lands `refs/lym/tags/archive/t2/1f515577c591`; the typed re-run deletes `refs/lym/heads/t2-weft`, keeps `refs/heads/t2` (unlanded, a correct absence), and clears the record; `status` lists `pending_teardowns: []`.
- Verdict: OK. The declined run is the sabotage control: the record is cleared only by the run with no failed step.

**P3 — focus 2, a tag kept local and a later `--remote` teardown**

- `lym fabric remove t3` without `--remote`: the hook log shows `refs/lym/tags/archive/t3/60dc9d5e80ad` pushed, `refs/lym/heads/t3-weft` kept.
  So a plain removal never leaves its archive tag local while origin is reachable; the focus's premise holds only for a failed push or for `DropLocal`, whose doc says its tags "stay local".
- `DropLocal` runs only under a takeover, which the sandbox cannot stage without a second machine's board entry.
  Stand-in: pair t4 pushed; in `t4-weft` a commit `a36fc7b3add2` adding `_lym/secret.txt`, tagged `archive/t4/a36fc7b3add2` by hand, then `reset --keep` back to origin's tip, as `DropLocal` leaves a store.
- `lym fabric remove t4 --remote`: envelope names no tag (`remote_handed_off: true`); hook log shows `refs/lym/tags/archive/t4/a36fc7b3add2` and `refs/lym/tags/archive/t4/584e56d206ae` created, then `refs/lym/heads/t4-weft` deleted.
  The commit holding `secret.txt` is now on origin for every clone.
  `lym fabric remove --help` says `--remote` deletes branches; it says nothing of pushing local archive tags.
- Verdict: WARN. By the focus's own failure test this is a tag pushed with no verb output or documentation naming it.
  `pushLocalArchiveTags` cannot tell a tag a killed teardown left from one `DropLocal` kept, since both carry `archive/<slug>/<sha12>`.
  Recorded under "Observations outside the incremental rule".

**P4 — focus 4, owner-archive tags read from two hubs**

- Blocker for the real push path: `ArchiveOwnerTips` runs only from the dobby's hand-over, `lym beam start`'s refusal of a worktree another machine owns, and the dobby's orphan-pair teardown.
  Staging it needs a board `prime_owner` of another machine, written only by the owner setter, and then `lym beam start`, which on any misread starts a dobby and an orch, a real session the cost declaration keeps out.
  So the push half is not driven; the tags below were placed by `git push` at the refs `pushArchiveTag` spells, which the focus marks as not proving the push path.
- Second hub: `lym fabric clone $O --into $H/c2` from inside the sandbox root, shortname `sbx`.
  Its records store holds `refs/tags/archive/t1..t4/*` from `refs/lym/tags/`, no code commit (`cat-file` of `4a97394c50b7`, `ef2bc44`, `e4f0d27` fails there), and its code store holds no tag.
- Placed `refs/lym/code-tags/archive-owner/root/66fbd1102ca9` (a code commit off `main`) and `refs/lym/tags/archive-owner/root/739cd1b9455c` (a records commit off `main-weft`).
- `lym adopt --archives root` in hub 1 and in c2: each lists exactly two tags, the code one with `side: code`, `branch: main`, the records one with `side: records`, `branch: main-weft`; no tag listed twice.
  Afterwards each code store holds only the code tag and each records store only the records tag; c2's records store lacks the code commit and its code store lacks the records commit.
- Verdict: WARN, read side OK; the push side is undriven for the blocker named above.

**P5 — focus 5, a plain `refs/tags/archive/` tag on a one-repository origin**

- Pair t5 pushed with an unlanded commit `66ba042ff1d6`; `git push origin <tip>:refs/tags/archive/t5/66ba042ff1d6` by hand, as an earlier binary left such tags.
- `lym fabric status`, `cleanup`, `prune`, `reconcile`: no output names `refs/tags/archive/t5/…` or any way forward for it.
- `lym fabric cleanup --apply --remote`: weft entries `main` only, `warp_skipped_reason` "the open pull requests could not be listed … unrecognized remote URL shape", so the warp sweep never runs on a sandbox origin (host-side, Coverage).
  `lym fabric prune --apply`: nothing.
- Third hub `lym fabric clone $O --into $H/c3`: its records store holds no t5 tag; its code store holds `refs/tags/archive/t5/66ba042ff1d6` (a code tag, by git's default tag following).
- `lym batten cancel t5 --remove`: hook log shows `refs/lym/code-tags/archive/t5/66ba042ff1d6` created before `refs/heads/t5` deleted, so no teardown deleted a branch whose only tag was plain.
  The plain ref stays on origin afterwards.
- Verdict: WARN. No teardown loses the commit, but by the focus's failure test the plain ref is a stray no verb names.
  Recorded under "Observations outside the incremental rule".

**P6 — focus 6, a relative origin URL, and the two-repository control**

- `git -C $P remote set-url origin ../../origin-bare`; `lym fabric add t6` (its worktree inherits the relative URL), commit, push; `lym batten cancel t6 --remove`.
  Hook log: `refs/lym/tags/archive/t6/e18d6c0a8405`, delete `refs/lym/heads/t6-weft`, `refs/lym/code-tags/archive/t6/f59544ee3d44`, delete `refs/heads/t6`, in that order.
  `git ls-remote` shows no `refs/tags/archive/` ref but the one P5 planted. URL restored afterwards.
- Control: two-repository hub `hub-3765637836` (`lym harden sandbox new`), code store URL made relative, pair c6, same cancel.
  Code origin holds `refs/tags/archive/c6/8566264a3644`, records origin `refs/tags/archive/c6/6d979ec83591`: the plain placement, differing from the one-repository result.
- Verdict: OK.

**P7 — focus 7, verbs with the records store's fetch refspec unset**

- Second one-repository hub `hub-3867511196` (shortname `sbx`, as hub 1), pair x1 pushed; `git -C <records store> config --unset-all remote.origin.fetch` (backed up under `.scratch/`).
- Driven: `fabric status` (prime, pair), `push`, `sync`, `pull` (pair, prime), `board sync`, `reconcile`, `cleanup`, `prune`, `add x2`, `pairs`.
  `push`, `pull`, `board sync`, `reconcile`, `cleanup`, `add` each fail with fabric's "holds the push refspec … so it is a one-repository records store, but no origin fetch refspec maps refs/lym/heads/; way forward: restore its fetch refspecs with …".
  `sync` spawns its push; the failure lands in `status`'s `failed_pushes` with the same text.
  `status`, `prune`, `pairs` succeed, reading no origin.
  No output carries a raw `gitrepo:` text.
- Static table. Enumerated by
  `grep -nE '\.(Fetch|Pull|UpstreamSHA|HasUnpushed|HasUnpulled)\(' internal/fabricengine/*.go internal/fabriccli/*.go | grep -v _test.go`:
  56 matches, one a comment (`pull.go:355`), so 55 calls.
  The grep cannot see a call through a method value or interface, a `FetchWithArgs` or raw `gitexec` fetch, or a call outside the two packages
  (`battencli/catchup.go:224` is one, a direct records-store `Fetch`).
  A records-store layout read precedes the call, in the function or a caller on every path, at:
  `bolt.go:319,320,325` (`pullFirst`, read at `bolt.go:305`), `recordsgit.go:300,331`, `pushclassified.go:149,210,227,347,350,365,552,555,580,626,824,924,927,965,968`,
  `coalesce.go:79`, `pull.go:539`, `pushanchored.go:231`, `hubpull.go:101,147`, `weft_verbs.go:376`, `takeoverwrite.go:271`.
  No read is needed on the code store at `mergeguards.go:47`, `handover.go:495`, `merge.go:567`, `pushclassified.go:398`, `parenttips.go:51,356`, `pull.go:610`, `pushanchored.go:395`, `codetipsettle.go:24`,
  nor at `clone.go:854` and `destroy.go:1751`, where the clone wrote the refspecs itself just before.
  A records or board call with no read before it on some path:
  `mergeguards.go:57` (`lym fabric merge`, `merge-in`); `pull.go:186` and `recordsgit.go:346` via `FastForwardPair` (batten's dobby reads);
  `codetipsettle.go:21,43` via `DobbyRuns`; `bolt.go:162` via `lym machine name`; `pushclassified.go:383` and `:769,772` on a code push over an unborn weft;
  `handover.go:178,193,289` and `takeoverwrite.go:129` only when the board pull before them is skipped by the skip env.
  Only the first is reachable from a `lym fabric` verb, so it is the one driven.
- Driven: pair y1 pushed; records fetch refspec unset; `lym fabric merge y1` in the prime staged the merge (`ok: true`, `merge_staged`) and named no layout gap, the only verb of this probe that did not.
  The records side is not a merge participant: its fetch brings nothing and the weft SHA is read from stale tracking refs into the merge record only.
  Verdict: WARN. Recorded under "Observations outside the incremental rule".

**P8 — focus 8, kills of pull, checkout, reconcile and clone**

Tools: `.scratch/killat.sh` kills a verb and its git child once that child appears;
`.scratch/slowpack.sh`, set as the stores' `remote.origin.uploadpack`, holds every fetch and ls-remote 4 s so the kill lands inside it.

- `fabric pull` in pair x1 with origin one commit ahead on both `refs/heads/x1` and `refs/lym/heads/x1-weft`:
  killed at its first `git fetch`, and again 9 s in. Both times the pair stayed on its old tips, clean, no ref added under either store's `refs/heads/`, no lock left;
  the re-run pulled both sides (`warp_fetched`, `weft_pulled`) and `status` reads clean.
- `fabric checkout`: it runs no fetch or push, and a kill swept from 30 ms to 700 ms always found it finished, so no mid-run window is reachable.
  Each switch moved both sides together (`feat`/`feat-weft`, `main`/`main-weft`); the code store gained no `-weft` branch.
- `fabric reconcile` killed at its first fetch: no ref changed; the re-run reports both pairs `already_healthy`.
- Clone read mid-flight: `lym fabric clone $O --into $H/c4` stopped (`kill -STOP`) as soon as its records store's `.git` appeared.
  `status`/`pairs` in the code store answer `not initialized here; run "lym fabric reconcile"`, and in the records store "not a warp worktree".
  After `kill -CONT` the clone finished and `status` reads clean.
- Clone killed at the same point (`$H/c5`, and again `$H/c6`): the hub holds the code and records stores, the records store with its namespaced refspecs, and no `_bolt`.
  `fabric status` and `fabric pull` print `not initialized here; run "lym fabric reconcile" …`.
  Typed `lym fabric reconcile` fails with the raw `gitrepo: git dir of …/_bolt: dotgit: not a git repository: …/_bolt/.git: stat …: no such file or directory`, twice.
  `board sync` names the missing `board.yaml` and reconcile again.
  The plain re-clone answers `hub already exists at …` with no way forward.
  `lym fabric clone $O --into $H/c5 --reset` rebuilt a working hub (`status` ok); no verb names it.
  Verdict: FAIL, a state whose printed way forward fails when typed. Recorded under "Observations outside the incremental rule".
- `handover.go` and `destroy.go`'s code-tip reset and hand-over paths need a takeover entry another machine pushed;
  `hubforge`'s topology seam is a test-support package no sandbox verb reaches, so they are not driven.

**P9 — focus 9, sabotage proofs, the seed, and two clones at once**

- Archive prefix split: the instrument is the origin hook's log of each ref a push names.
  It shows `refs/lym/code-tags/archive/<slug>/…` as the code tag's destination in P1, P5 and P6.
  The two-repository control lands `refs/tags/archive/c6/…`, so the split is the branch taken, not a constant.
  A sabotage by unsetting the records refspec before a discard (`lym batten cancel x1 --remove`) never reached the split: batten's own origin reconcile refused first (`battencli: origin reconcile of …`), nothing was tagged or deleted.
  With the refspec restored, the control run pushed `refs/lym/code-tags/archive/x1/a99ab0c520e8` and `refs/lym/tags/archive/x1/05681b4717af` and cleared its record.
- Pending record's tags: the record file is the instrument; P2's declined push kept `weft_tip` and printed the hand push, and P1's killed child kept the record that the re-run cleared.
- Layout read before the board pull: P7's `board sync` with the refspec removed answers fabric's layout error, not a raw `gitrepo:` text.
- Seed killed in its records push: a code-only bare origin `seedo.git` under the sandbox root, hook holding `refs/lym/heads/main-weft`.
  `lym fabric clone --single-repo --shortname sbx` killed there; origin keeps only `refs/heads/main`.
  `board sync` and `reconcile` name "run the same `lym fabric clone --single-repo` again with `--reset`"; typed, it seeded `refs/lym/heads/main` and `main-weft`, and `board sync` then answered `ok`.
- Seed killed at its board push (`seedo2.git`, hook holding exactly `refs/lym/heads/main`): origin keeps `refs/lym/heads/main-weft` and no board.
  `board sync` names the same `--reset` re-seed; a plain join clone refuses with "a one-repository seed stopped before its board landed; way forward: run `lym fabric clone --single-repo --shortname <shortname> <url>`".
  Typed as printed from an empty `s4`: it seeded the board and replaced `main-weft` (`f75c384`), leaving the stopped hub `s3` behind with a records tip origin no longer holds.
- Two clones at once (`.scratch/race.sh`): each prime of a pair of clones upserts a note and runs `board sync`.
  Sequential control on hub 1 (prime and `c2`): both notes on origin.
  Parallel on hub 1 and, at the same time, on hub 3 (prime and its clone `c7`), both shortname `sbx`: all four notes on origin, every sync `ok`, no plain `refs/heads/` ref added.
  A second parallel run on hub 1: both notes on origin.
  The hook log of `refs/lym/heads/main` shows one unbroken chain of old→new tips with no push naming a stale old tip, so no lease lost; the race was serialised by the pull-first board push.
  A losing lease stays undriven.

## Coverage

- Not driven: the owner-archive push path (P4), the hand-over and code-tip reset paths of `handover.go` and `destroy.go` (P8), a lease that loses (P9), and a kill inside `fabric checkout`, which finishes in under 30 ms.
- `cleanup --apply --remote`'s warp sweep never runs on a sandbox origin: it lists open pull requests through GitHub and skips on a local path URL. That is host-side behaviour.
- The method's limits: the hook log proves which ref a push names, not that nothing else wrote origin.
  The race found no contention, so it shows the serialised path only.
  `DropLocal` was stood in for by a hand-made tag (P3).
  The focus 7 table rests on caller tracing; only the `lym fabric` path among its gaps was driven.

## Teardown check

```
$ lym harden sandbox clear
{"ended":[],"ok":true,"root":".../harden-fabric-single-repo-verbs/.lym/harden-sandbox"}
$ lym harden sandbox list
{"hubs":[],"ok":true}
$ pgrep -af 'harden-sandbox'
(only the shell running this check)
$ pgrep -af 'lym (reed|fabric|board)'
lym reed watchdog --hub-path /home/knatte/Code/scribe-LYMHUB …
lym reed watchdog --hub-path /home/knatte/Code/loomyard-LYMHUB …
$ ls .lym/harden-sandbox
hubs.json
hubs.json.lock
$ git -C . worktree list
(the hub's worktrees: loomyard, beam-pr-describes-itself, code-bouncer-asks-parent, harden-fabric-single-repo-verbs, note-depends-on)
$ git -C . status --short
(empty)
$ git -C . stash list
(empty)
```

The two watchdogs belong to the real hubs. Every scratch file was removed by its literal path; `.scratch/` is empty.

## Focus departures

- Focus 4, "stage a second machine's owner entry … and let the real path push its tags, never tags made by hand":
  the tags were placed by hand at the refs `pushArchiveTag` spells, since staging the real path risks starting a real session (P4 names the blocker).
- Focus 8, "drive the code-tip reset and hand-over paths … as far as hubforge's topology seam allows":
  the seam is test-support code no sandbox verb reaches, so these paths stay undriven.
- Focus 2's repro, "remove it without `--remote` so a local archive tag stays":
  a plain removal pushed its tag, so the kept tag was made by hand as `DropLocal` makes one (P3).
- The rubric's "record every finding" against the format's incremental rule:
  the five defects are recorded in prose under "Observations outside the incremental rule" and not in the frontmatter.
  The judge should read O1 as an open MEDIUM.
