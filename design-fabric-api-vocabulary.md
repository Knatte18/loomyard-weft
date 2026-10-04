# Fabric exports without warp/weft in their names

`fabric-api-vocabulary` · Notes · chore

Seen 2026-10-04 in `pattern-init`: extending the fabric-vocabulary scan to test files outside fabric flagged 83 files, and 38 of them only name fabric's own exports that carry the words, e.g. `hubforge.Hub.PairWarpWorktree` and `WEFT_SKIP_GIT`.
`pattern-init` exempts that closed set of 13 identifiers rather than renaming them.
Idea: rename those exports to neutral names so the vocabulary stays inside fabric, then shrink the exemption set to nothing.
