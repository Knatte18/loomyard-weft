# Card 10 — Checkout and adopt's drop-local refuse another anchor

**Edit:**
- `internal/fabricengine#Topology.Checkout`
- `internal/fabricengine#DropLocal`
- `internal/fabricengine_test#TestDropLocal`

**Create:**
- `plan:internal/fabricengine_test#TestCheckout_RefusesAnotherAnchor` -> `func TestCheckout_RefusesAnotherAnchor(t *testing.T)`

**Uses:**
- `plan:internal/fabricengine#treeAnchor`
- `plan:internal/fabricengine#weftAnchorStop`
- `plan:internal/fabricengine#refuseEmptyOwnMarker`
- `plan:internal/fabricengine#ErrAnchorChange`
- `internal/lyxcwd#NamedWorktree`
- `plan:internal/hubforge#ReanchorPair`
- `internal/fabricengine#Topology.switchOrForkWeft`
- `internal/fabricengine#resolveWeftBranch`
- `internal/hubforge#CloneSecond`

**Intent:**
D10's checkout and drop-local refusals; each runs before anything is switched, tagged or moved.

`Topology.Checkout` first refuses an empty own marker of its weft (`refuseEmptyOwnMarker`).
Then it finds the target weft branch `RecordsBranchName(branch)`'s tip from every source `switchOrForkWeft` switches to without forking: the local branch, else origin's remote-tracking ref, else origin itself through the same probe `resolveWeftBranch` runs (`remoteHeadTip`), in which case it fetches that tip into the remote-tracking ref only, creating no local branch; it reads that tip's anchor with `treeAnchor` (fallback: the worktree's own `AnchorRel`) and refuses, wrapping `ErrAnchorChange`, when it differs from `l.AnchorRel`.
An unreachable origin is the error `switchOrForkWeft` already returns for it, raised before any switch.
The refusal names `lyx fabric anchor --set <target anchor>` and then the checkout again, and the cost: the `--set` commits the new marker on the branch being left, shared by every machine that checks it out, whose pulls and pushes then stop until each integrates.
When `<target anchor>` is not a directory git tracks at the current warp `HEAD` (`git ls-tree -d HEAD -- <anchor>`), it names first updating the warp checkout by hand to a commit that tracks it, such as merging the target warp branch into the current one or committing the anchor directory on it, with that update's cost: a code commit on the branch being left, shared by every machine that checks it out; then the `--set`, then the checkout.
A target weft branch that does not exist is forked from the current one, inherits its marker, and is never refused.

`DropLocal`, for the records target of the pair, refuses an empty own marker, then compares the checked-out anchor with `treeAnchor` of origin's tracking tip it would reset to through `weftAnchorStop` with `reset` set, so the comparison ignores ancestry: a local weft ahead of origin with a re-anchor not yet pushed differs from origin's tip as well; a difference refuses before any tag or move, naming `lyx fabric anchor <slug> --set <origin's anchor>` and then the adopt again (`lyx adopt <slug> --drop-local`).

Tests:
`TestCheckout_RefusesAnotherAnchor` (tier1, hubforge) first commits `sub/.keep` in the prime's warp before it adds pairs, so `sub` is tracked in every pair's warp: a pair whose target branch's weft tip was re-anchored at `sub` (made with `hubforge.ReanchorPair` on a second pair checked out on that branch, then switched away) refuses the checkout naming `lyx fabric anchor --set sub` and the branch-left cost and no warp update, with both sides still on their branches and nothing rewired; a target re-anchored at `only-there`, which only the target warp branch tracks, makes the message name the warp update first; a target branch pushed from a second clone (`hubforge.CloneSecond`) at another anchor and never fetched here is refused the same way, with no local weft branch created; a checkout to a new branch forks and succeeds; an empty own marker refuses naming `.lyx-anchor`.
`TestDropLocal` gains rows: origin's tip of the pair's weft carries another marker, and the drop refuses naming `lyx fabric anchor` and `lyx adopt`, with no tag and both branches unmoved; the local weft ahead of origin with a re-anchor not yet pushed refuses the same way; an empty own marker refuses the same way.

**ImpactSummary:** two entry points gain refusals ahead of their first mutation; their callers (`fabriccli` checkout and `adoptcli`'s drop-local) need no change, and a one-anchor hub never reaches either refusal.
