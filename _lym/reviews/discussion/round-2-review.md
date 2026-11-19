---
verdict: BLOCKING
findings:
  - id: F14
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:42
    summary: "D1's fixed board branch `main` contradicts fabric's existing rule that the board's branch is the code store's primary branch, so on a code default branch other than `main` the orphan sweep and `resolveManagedBranch` treat `<prime>-weft` as deletable, the `--single-repo` bind seeds the board on the wrong branch, and a second machine's clone orphan-creates an empty board"
    fix: "In D1, drop the claim that local geometry is a two-repository hub's unchanged and name the one place it differs: the board worktree's local branch is D1's board constant `main`, not the code store's branch. State that the clone, both the one-URL path and `--single-repo`, creates or adopts the board on that constant (`ensureBoardWorktree` is handed the board name, never `warpBranch`), and that the `--single-repo` records store is made by `git init` with D1's refspecs and a fetch, as the one-URL path's is, never by the two-repository weft clone. State the source `primaryWeftBranch` (internal/fabricengine/cleanup.go:337) reads for the prime's records branch in a one-repository hub, a durable one that a later `lym fabric checkout` on the prime cannot move, so the orphan sweep and `resolveManagedBranch` (internal/fabricengine/destroy.go:828, 847) never treat `<prime-branch>-weft` as an orphan; in a two-repository hub it must still answer what it answers today. In Acceptance criteria, add: a `--single-repo` clone of an origin whose code default branch is `master` puts the board at `refs/lym/heads/main` and the prime's records branch at `refs/lym/heads/master-weft`, a one-URL clone of it on a second machine checks out that board with its content, and the orphan sweep in that hub keeps `master-weft`."
prior:
  - id: F1
    status: resolved
    answer: "D3 now places the seed push at CloneAndWire's board-push step, after the Bolt commit and before junction wiring, as one fabric verb that is today's board push in a two-repository hub, and its rationale names the records push as added behaviour."
  - id: F2
    status: resolved
    answer: "The `.lym-anchor` and `_lym` arm applies with `--single-repo` only, and a criterion pins that a two-repository weft with the legacy board marker clones as today; the fixer's reason for not taking either proposed option holds."
  - id: F3
    status: resolved
  - id: F4
    status: resolved
    answer: "D1 names the board branch and the prime's records branch, D2's HEAD answer and D3's probe use the board name. The fixed name breaks existing readers of the board branch, which is raised as the new finding F14."
  - id: F5
    status: resolved
  - id: F6
    status: resolved
  - id: F7
    status: resolved
  - id: F8
    status: resolved
  - id: F9
    status: resolved
  - id: F10
    status: resolved
  - id: F11
    status: resolved
  - id: F12
    status: resolved
  - id: F13
    status: resolved
---

### [BLOCKING] D1's fixed board name `main` breaks the existing readers of the board's branch

**Location:** `_lym/discussion/decision-record.md:42` (D1), with D3 at lines 109–113.

**Issue:** Kept from bound-breaker finding 1, merged with interactions finding 1, walkthrough F-W1 and existing-paths finding 2, which describe the same defect from three sides.
The round-1 fix for F4 made the board branch the fixed name `main`, while the prime's records branch stays `<prime-branch>-weft`.
Today fabric ties the two together.
`CloneHub` builds the board with `ensureBoardWorktree(weftPath, warpBranch, boardDir)` (`internal/fabricengine/clone.go:328`), and `ensureBoardWorktree` adopts `origin/<warpBranch>` or orphan-creates it (`internal/fabricengine/boardweft.go:24-51`).
`primaryWeftBranch` (`internal/fabricengine/cleanup.go:337-351`) derives the prime's records branch from the board's checked-out branch, and its doc says a wrong answer loses data.
The orphan sweep (`cleanup.go:134`) and `resolveManagedBranch` (`destroy.go:828, 847`) read it.
Take a code repository whose default branch is `master`:

- `--single-repo`, seeded "as a fresh two-repository bind seeds its weft", creates the board on local `master`, while D3 pushes it leased to `refs/lym/heads/main`;
- a second machine's one-URL clone looks for `origin/master` in the records store, finds none and orphan-creates an empty board;
- in a hub whose board is on `main`, `primaryWeftBranch` answers `main-weft`, so the sweep can enumerate the prime's `master-weft` records line as an orphan.

D1 states that the local geometry is unchanged, and D3 states that the clone wires the hub as a two-repository clone does, so neither the plan writer nor the fixer is told that these readers must change.
Every acceptance criterion passes over a fixture whose default branch is `main`, so no criterion would catch it.

## Focus verdicts

- D3's clone sequence: the seed push's stated place, `CloneAndWire`'s board-push step after the Bolt commit, matches `internal/fabriccli/clone.go:73-96`; `fabriccli` reaches it through a mark on `CloneHub`'s result and needs no layout branch outside fabric.
  The seed records branch holds the anchor-marker commit at that step, since the module-config commit comes later and is not pushed, as in a two-repository clone.
  The two-repository bind is described accurately except for the board branch, which is F14.
- The legacy two-repository weft: no refusal arm refuses it.
  Without `--single-repo` only the `lym-bolt` arm applies, and such a weft holds no `lym-bolt` and no `refs/lym/heads/main`, so it takes today's path and `CloneHub`'s legacy adopt (`clone.go:365-371`).
- The branch names: D2's remote `HEAD` answer and D3's probe both use the board constant, and a renamed code default branch cannot hide the board on origin.
  One source does not name both branches, and the fixed board name breaks the local readers, which is F14.
- Seed pushes and rerun: a rerun replaces a leftover records branch, since D3 creates the branch fresh and the anchor takes the create path, so `adoptRecordedAnchor` refuses no `--subpath`.
  Concurrent clones are bounded at push time by the two leases, and every failed seed push, refused, unreachable or on a failed lease, takes the stated teardown.
  The residual gaps are below the incremental bar, listed below.
- The teardown delete: D4 tells an absent namespaced branch from a deleted one with an `ls-remote` of the full name before the push.
  The git behaviour is pinned for the URL push `gitrepo` runs (`internal/gitrepo/pushdestination.go:65-68`), with the command, its output and the git version.

## Below the incremental bar

This round admits only BLOCKING new findings.
These defects are real but below that bar, so they are listed here and carried by no finding:

- MEDIUM, D3 lines 142–153 (walkthrough F-W2, bound-breaker 4, existing-paths 3): D3 says the precondition check is one `git ls-remote`, but its `.lym-warp`, root `.lym-anchor` and `_lym` arms need the default branch's tree, which today only `probeWeftBinding`'s staged clone reads (`internal/fabricengine/recordsprobe.go:60-120`).
  The walkthrough rated it BLOCKING; it is MEDIUM, because the refusals and the criterion that tests them are stated plainly and only the mechanism is misnamed.
- MEDIUM, D3 (existing-paths 1): a one-URL `--reset --subpath` clone of a one-repository hub passes `checkSubpathBeforeReset`, which stages the URL with default refspecs and misses `refs/lym/heads/<prime>-weft` (`internal/fabricengine/resetsubpathcheck.go:26-100`).
  `resetHub` then removes the hub before `adoptRecordedAnchor` refuses the mismatch.
  D3 never mentions `--reset`.
- MEDIUM, D3 lines 117–119 (bound-breaker 2): the records-branch replace cannot tell a failed attempt's leftover from a concurrent clone still in flight; a killed or cut-off second clone can leave the first hub's board beside the second clone's anchor marker.
- LOW, D3 line 161 (bound-breaker 3): when origin turns unreachable after the records push lands, the restore cannot run, so the bound "leaves the records branch as it found it" fails; a rerun replaces the leftover.
- LOW, D4 lines 181–183 (interactions 2, bound-breaker 5 and 6): the `ls-remote` before the plain delete changes a two-repository hub's delete path, which Scope Out's "beyond D7's reverts" does not allow, and the check races a concurrent delete into a misattributed removal record.

## Rejected

- None of the forks' findings was a false positive; the non-BLOCKING ones are listed above instead.

## Notes

- `lym board get single-repo-records-branches` answered `invalid json`, so the board entry was not read; the support log's account of the original body, read from `_bolt` commit 5a3ab0c956, was taken as given.
