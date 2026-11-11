# adopt --drop-local fails on an untracked lym file in the weft

`drop-local-untracked-weft` · Note · bug, fabric

## Seen (2026-11-11, Neve)

`lym adopt root --drop-local` archived the local tip of `push-notice-anchor-dir-weft` as `archive/push-notice-anchor-dir/5e8b95669e68`,
then failed to move the branch to origin's tip: `git reset --keep` refused because the untracked `_lym/reviews/code/generation-record.json` would be overwritten.
The file is one lym writes in the weft, so the operator has no clean way forward, and the adopt ended partial, holding back every other run under root.

## Idea

Once the tip is archived, drop-local moves the weft with a reset that also clears untracked lym files the target commit tracks, since the archive tag keeps the old state.
