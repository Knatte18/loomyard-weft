# stencilstore: a pair branched before the last deploy warns source-ahead and asks for a deploy

`stencil-drift-behind-main` · Note · bug, stencil

Imported from [issue #445](https://github.com/Knatte18/loomyard/issues/445).

Seen: `lyx loom status` in each of three task pairs (plan-glyph-precision, shuttle-state, reed-view) logs
`stencilstore: board copy has drifted from worktree source; the board copy is untouched and a binary older than the source deployed it -- run a production deploy (update-plugins.sh)` with `stencil=webster-template-master class=source-ahead`.

The pairs branched from main at fd5b0f58e; main and prod then moved to c0bf19c54, which changed `contracts/stencils/webster/webster-template-master.md`.
The installed binary is c0bf19c54 and the board copy matches c0bf19c54, so the pair's source is behind the board copy, not ahead, and a deploy fixes nothing.
The prime logs no warning.

Expected: a worktree whose source is older than the board copy (a pair that branched before the last deploy) is not classified source-ahead and does not ask for a production deploy; at most it notes that the pair is behind main.

Workaround: none needed; the warning is ignored until the pairs sync with main at Finalize.
