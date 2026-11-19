---
dispositions:
  - id: F14
    disposition: fixed-differently
    reason: "Carried out every part of the proposed fix except the fetch into the `--single-repo` records store. The fix key had that store made by `git init` with D1's refspecs and a fetch, as the one-URL path's is. A fetch maps a leftover `refs/lym/heads/<prime>-weft` to `origin/<prime>-weft`, and the fresh bind's `suffixWeftPrimaryBranch` then adopts it (internal/fabricengine/clone.go:515-523). The clone would then take `adoptRecordedAnchor`'s path and refuse a rerun with another `--subpath`, which D3 and its rerun criterion forbid. The `--single-repo` store is therefore made by `git init` with D1's refspecs and fetches nothing."
---

# Round 2 fixer-report — discussion

## Changes

- F14 (fixed differently, see the frontmatter reason), in `_lym/discussion/decision-record.md`:
  - D1 now says the local geometry matches a two-repository hub's in every part but the board's branch.
    The board worktree is on the board constant `main`, never on the code store's branch.
  - D1 records the prime's records branch once at clone, in the records store's git config key `lym.primeRecordsBranch`.
    In a one-repository hub, `primaryWeftBranch` reads that key, so neither the orphan sweep nor `resolveManagedBranch` can treat `<prime-branch>-weft` as an orphan, and `lym fabric checkout` cannot move it.
    A missing key fails closed.
    A two-repository hub reads the board's branch, as today.
    The config key joins the named constants.
  - D3, one-URL path: the board is checked out on D1's board branch, and `ensureBoardWorktree` is handed the board name.
    The board therefore adopts `origin/main`, mapped from `refs/lym/heads/main`.
    The path records the prime's records branch.
  - D3, `--single-repo`: the records store is made by `git init` with D1's refspecs and fetches nothing, never by the two-repository weft clone.
    The board is orphan-created on D1's board branch, and the path records the prime's records branch.
  - Acceptance criteria: one new criterion uses an origin whose code default branch is `master`.
    A `--single-repo` clone puts the board at `refs/lym/heads/main` and the records branch at `refs/lym/heads/master-weft`.
    A second machine's one-URL clone checks out that board with its content.
    The sweep keeps `master-weft`, including after `lym fabric checkout` moves the prime.
- `_lym/discussion/support-log.md`:
  - A Round 2 section.
  - A rejected alternative: reading the prime's records branch from `origin/HEAD` or from the records worktree's branch.
  - Ledger entry Q11.
- Self-check fixes:
  - D1's checkout sentence now names the key.
  - D1's missing-key refusal names its way forward.
  - The Goal, D3's rationale and the support log's geometry claim name the board-branch difference.
  - D5 tells the removed `lym.primeBranch` apart from the new key.
  - The `--single-repo` fetch is dropped (F14's reason).

- Gate repair: the decision record's citations of the config keys carry the gate's markers.
  `lym.primeRecordsBranch` is marked ` (new)` in D1 and D5, and `lym.primeBranch` is marked ` (removed)` in D5.

## Self-check

- fix-claims, MEDIUM: D1 said checkout "writes no config", but checkout writes upstream config. Fixed: the sentence now says checkout never writes the key.
- fix-proposal, MEDIUM: the proposed fetch into the `--single-repo` store makes the fresh bind adopt a leftover records branch and breaks the rerun criterion. Fixed differently from the proposal: the store fetches nothing.
- fix-way-forward, LOW: the missing-key refusal named no way forward. Fixed: it names `git config` of the key to `<prime-branch>-weft`.
- fix-consistency, LOW: the Goal still said "wired exactly as" the weft. Fixed.
- fix-consistency, LOW: D3's rationale did not name the departures from the weft bind. Fixed.
- fix-consistency, LOW: the support log said the "local geometry identical". Fixed.
- fix-consistency, LOW: D5's removed `lym.primeBranch` could be mistaken for the new key. Fixed: D5 names them as separate keys.

## Disputed

None.

## Deferred

None.

## Rulings

None.
