# Shared object store triggers auto-gc on nearly every git write

`shared-store-auto-gc` · Note · enhancement, fabric

## Seen

The hub's shared object store (`loomyard/.git`, used by the prime and every pair) holds about 8,450 loose objects, above git's default `gc.auto` threshold of 6,700.
They survive each repack, most likely because they are unreferenced and kept until the two-week prune expiry, so nearly every commit, merge or fetch in any worktree starts a background auto-gc.
On 2026-10-08 new packs appeared at 16:40, 18:07, 19:34, 19:52, 20:00 and 20:42, and the 20:42 one broke a Publish merge-in (#468).

## Idea

lyx keeps the shared store below the auto-gc threshold, for example by pruning unreferenced loose objects at landing, or sets `gc.auto` for the hub's store so repacks run at known points rather than under a live step.
