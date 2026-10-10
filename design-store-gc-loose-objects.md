# Hub store gc may repack at every teardown

`store-gc-loose-objects` · Note · enhancement, fabric

Since `git-robustness`, the hub store's auto gc is off and Remove runs `git gc --auto` at teardown.
Loose objects younger than the prune expiry survive each repack, so `gc --auto` can find the loose count over its threshold at every teardown and repack without lowering it.
Seen in the PR-Gate review of PR #483.

Idea: run `git prune --expire` now and then, such as at landing, so the loose count falls and `gc --auto` stops repacking every time.
