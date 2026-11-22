---
verdict: APPROVED
prior:
  - id: G1
    status: resolved
    answer: "Drove a discard with the code store's origin URL spelled as file://<bare path> and as a symlink to it; both left refs/lym/code-tags/archive/<slug>/<sha12> on origin, deleted the task branch there, and left no ref under refs/tags/."
  - id: G2
    status: resolved
    answer: "With remote.origin.fetch unset in the records store, reconcile, push, pull, add, remove, cleanup --apply and board sync each answered with the records-store message and its restore commands; no gitrepo resolve-upstream text appeared."
  - id: G3
    status: resolved
    answer: "Records half driven: a killed remove --remote left a pending record that status and prune named with the tag push command; the re-run pushed the local tag of the unpushed records commit and archived origin's tip before deleting the records branch. The discard half was not driven."
  - id: G4
    status: resolved
    answer: "After a discard and a remove --remote, origin holds records tags under refs/lym/tags/ and code tags under refs/lym/code-tags/; a second clone's records store holds only records tags, and its code store holds no records commit."
---

## Probes

Hub: `lym harden sandbox new --single-repo`, driven with the printed `.dev-bin/lym`.
Control on the two-repository layout was not driven (see Coverage).

- Focus 1, archive tag of a discarded branch, URL spellings.
  `fabric add t1`, a real commit, `fabric push`, `git remote set-url origin file://<bare>`, `batten cancel t1 --remove`.
  Origin afterwards: `refs/lym/code-tags/archive/t1/579c98624627` and `refs/lym/tags/archive/t1/4e71e3d9d115`, no `refs/heads/t1`, no `refs/tags/`.
  The same drive with a symlinked origin URL (`t2`) gave the same layout once the detached remote job finished.
  The relative spelling was not driven. OK.
- Focus 2, prefix split.
  `fabric clone <url> --into <dir>` of that origin: records store holds only `refs/tags/archive/t*` of records commits, code store holds no records commit.
  `adopt --archives` over real owner-archive tags was not staged. WARN.
- Focus 3, killed teardown, records half.
  A local records commit `6f61f1fe153f` on `t3-weft`; a `pre-receive` hook sleeping on `refs/lym/heads/t3-weft` then declining; `fabric remove t3 --remote`; `kill -9` of the `remote-teardown` child.
  `fabric status` and `fabric prune` listed the pending record with `weft_tip` and the tag push command as way forward.
  Re-running `fabric remove t3 --remote` pushed `refs/lym/tags/archive/t3/6f61f1fe153f` and `.../f4b03b0f4ca4`, deleted `refs/lym/heads/t3-weft`, and cleared the record; `git cat-file -t 6f61f1fe153f` in origin answered `commit`.
  The discard half was not driven. WARN.
- Focus 6, layout read.
  `git config --unset-all remote.origin.fetch` in the records store, then every verb.
  `reconcile`, `push`, `pull`, `add`, `remove`, `cleanup --apply` and `board sync` answered with the restore commands; `fabric sync` spawned a detached push that `fabric status` then listed under `failed_pushes` with the same restore commands; `fabric checkout main` and `prune --apply` succeeded without reading upstream; `adopt --archives` needs a board entry id, not a hub path.
  Typing the printed restore commands made `reconcile`, `pull`, `push`, `add` and `board sync` succeed. OK.
  The call-by-call enumeration of `Fetch`, `Pull`, `UpstreamSHA`, `HasUnpushed` and `HasUnpulled` was not made.
- Focus 7, legacy plain `refs/tags/archive/` tag.
  A tag made by hand on origin; `status`, `cleanup`, `cleanup --apply --remote`, `prune` and `reconcile` printed no text naming it.
  A clone left it in the code store and put no such tag in the records store.
  No reader names it. WARN (recorded under Rejected: an incremental round raises only BLOCKING findings).
- Focus 8, killed seed.
  A bare origin with a `post-receive` hook sleeping after `refs/lym/heads/main-weft`; `fabric clone --single-repo --shortname sx <url> --into <dir>` killed mid-seed.
  A plain clone of the mid-seed origin answered with the seed way forward for the one-repository layout.
  That printed command, typed exactly from an empty directory, seeded the hub and landed `refs/lym/heads/main`.
  `--into` naming a missing directory answered `create it, or pass --into an existing directory`, with no git text. OK.
- Teardown check: `sandbox clear` ended nothing and emptied the root, `sandbox list` is empty, `.lym/harden-sandbox` holds only `hubs.json` and its lock, `git status` is clean, no stash.
  The two `lym reed watchdog` processes listed belong to the real hubs, not the sandbox.

## Rejected

- Plain `refs/tags/archive/` refs an earlier binary left on origin are named by no reader verb, and a clone fetches them into the code store.
  LOW, not raised: an incremental round raises only BLOCKING findings.
- A killed `fabric clone` leaves its `git push` child running until the origin hook ends; the push was not reaped by `kill -9` of lym.
  Not raised for the same reason.
- `fabric remove <slug> --remote` leaves `refs/heads/<slug>` on origin; no remove verb deletes the code branch, so it is read as by design.

## Coverage

Not driven: the discard half of the killed teardown, a keep-local tag drive (focus 4), a failed-step pending record without a kill (focus 5), the two-repository control, the relative URL spelling, `adopt --archives` over real owner tags, concurrent claims on a second hub, sabotage proofs of each fix, and the regions of focus 9.
The records-half drive was shown to enter the mechanism by the hook log of the held push and by the pending record the kill left.
