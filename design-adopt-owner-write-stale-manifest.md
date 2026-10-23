# adopt's owner write fails on a stale local render manifest after another machine removed design files

`adopt-owner-write-stale-manifest` · Task · bug, board · running · Darn-Write

Imported from [issue #844](https://github.com/Knatte18/loomyard/issues/844).

## Seen (orch, 2026-10-23, OSL-1046, lyx e634efb9)

`lyx adopt plan-card-rules` on a machine whose last local board render was two days old failed at the owner write:
`gitrepo: git add: git add -- board.json README.md design-...md ...: exit 128: fatal: pathspec 'design-batten-abort-live-run.md' did not match any files`.
The other machine had since removed that entry and its design file on Bolt, and this machine had pulled Bolt.
`renderOwnerWrite` in `internal/boardengine/ownerset.go` adds every name in the local, gitignored `.board-rendered.json` that the new render no longer writes, so a file already gone from both disk and git reaches `git add` and fails it.
The failed write left `board.json` and `README.md` modified and uncommitted on Bolt, and the pair was already added (partial).

## Workaround

Re-run `lyx adopt <slug>`: the failed render rewrote the manifest, so the second run's path list holds only existing files, and it committed and pushed the owner write.

## Expected

The owner write commits only paths that exist or that git tracks, so a stale manifest from before a pull never fails it.
