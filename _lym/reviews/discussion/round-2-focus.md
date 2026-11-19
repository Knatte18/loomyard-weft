---
round: 2
exclude_lenses: []
focus:
  - "D3's clone sequence in `_lym/discussion/decision-record.md`: `fabriccli.CloneAndWire` calls `CloneHub`, and the board's `.lym-warp` and `.lym-shortname` are committed only in the Bolt commit after `CloneHub`. Settle whether the stated place of the seed push and its content can run without a layout branch outside fabric, and whether the two-repository bind path is described accurately."
  - "D3's refusals against a one-URL clone of an existing two-repository weft: `CloneHub` still adopts a legacy hub-wide `.lym-anchor` at the board root, and `retireBoltAnchorMarker` removes it only on reconcile. Settle whether any refusal arm refuses such a weft, given Scope Out promises no behaviour change to a two-repository hub."
  - "D1 to D3: the board branch's name and the prime's records branch name under `refs/lym/heads/`. Settle whether one source names both, whether D2's remote `HEAD` answer and D3's `ls-remote` probe use it, and whether a renamed code default branch can hide the board."
  - "D3 seed pushes and rerun: `suffixWeftPrimaryBranch` checks out a fetched `origin/<prime>-weft`, and `adoptRecordedAnchor` refuses a different explicit `--subpath`. Settle whether a rerun after a partial seed push adopts or replaces a leftover records branch, whether concurrent `--single-repo` clones are bounded at push time, and whether every failed seed push, refused or unreachable, takes the stated teardown."
  - "D4 teardown delete: `gitrepo.Repo.DeleteRemoteBranch` pushes to the origin URL, and its `(false, nil)` answer for an absent ref feeds the destruction gate in `internal/fabricengine/destroy.go`. Settle whether the decision tells an absent namespaced records branch from a deleted one, and whether its stated git behaviour is pinned for the path `gitrepo` takes."
---

Every lens stays: the gating areas above are open for all of them, and the new text the fixer writes there has not been seen by a fresh reviewer.
