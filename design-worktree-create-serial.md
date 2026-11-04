# Worktree-Create runs one at a time and spends minutes on network round trips

`worktree-create-serial` · Note · bug, fabric, batten, dobby

## Seen (operator and hub orch, 2026-11-04)

Seven tasks were released together on the loomyard prime; five were ready at once and dobby (max_runs 8) started all five.
The operator found the start very slow.
Over about three minutes the five runs left Worktree-Create one after another, never together:

- `beam-pr-lifecycle` reached Seed-IDE first, while the other four still sat in Worktree-Create.
- `dobby-zombie-children` then sat at Open-Window for over a minute.
- `shuttle-launch-delivery` left Worktree-Create about two minutes in; `drift-capture-and-migration` and `drive-reports-untracked` were still in Worktree-Create after that.

Meanwhile `lym tender status` showed network operations in flight only (several `remote-refs` and a `fetch` per slug, none under a parent lock), and a `lym fabric … push` of the prime's weft for `beam-pr-lifecycle` ran over two minutes, its `git send-pack` to `loomyard-weft.git` alone about a minute.

## Idea

- Find what serializes Worktree-Create across runs (a fabric lock, the weft push, or the network ops), and let independent creates run side by side.
- Cut the network round trips per create: one fetch and one remote-refs read per create, and no synchronous weft push of the prime on the create path.
- Find why Open-Window takes over a minute.
