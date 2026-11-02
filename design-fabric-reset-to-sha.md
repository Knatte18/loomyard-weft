# fabric reset: reset warp to a SHA and re-anchor the weft trailers

`fabric-reset-to-sha` · Note · feature, fabric

## Seen

Weft commits carry `Warp-SHA` trailers linking them to warp commits, and the correspondence index is rebuilt from those trailers.
A `git reset` of warp by hand leaves the weft's newest trailers naming warp commits that are gone, so the correspondence no longer holds.
`lym fabric pull` already handles the same break when the rewrite comes from origin: it re-anchors the weft to the nearest surviving `Warp-SHA` with an empty commit carrying the new trailer, then resets warp (`internal/fabricengine` package documentation, the `Fabric.Pull` section).
No verb does this for a reset made on this machine: `MergeAbort` covers only the uncommitted merge window, so a committed merge or any other local warp commit can only be undone by hand, outside fabric.

## Idea

A verb `lym fabric reset <warp-sha>` resets warp to a SHA visible from the current branch and re-anchors the weft the way `Pull` does for an upstream rewrite, through the destruction gate, so no uncommitted or unpushed work is lost unannounced.
It reuses `Pull`'s re-anchor path rather than a second mechanism.

## Open

- Whether a reset of a pushed commit also force-pushes, or leaves that to a separate push with its own guard.
