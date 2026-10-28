# Burler fork audit reads a listing git branch as read-only

`fork-audit-branch-list` · Task · bug, burler · running · Drift-Baseline

## Burler fork audit counts a listing git branch as git-mutating (was note `fork-audit-git-branch-list`)

Imported from [issue #892](https://github.com/Knatte18/loomyard/issues/892).

Seen: validator-checks failed at Discussion-Burler-Review (round 2 attempt 2, 2026-10-28T11:02Z) because a review fork ran `git branch -a --contains 71a46b11`. internal/burlerengine/cluster.go lists `branch` in mutatingGitSubcommands and only `git tag` gets a list-mode exception, so a read-only branch listing fails the whole review round.

Workaround: the orch resumed the run with `lym loom resume`, which re-runs the round; a fork that lists branches again fails it again.

Expected: `git branch` counts as mutating only when it creates, deletes, renames, copies or moves a branch or sets its upstream; with no operand, or with list-mode options (-a, -r, -l/--list, --contains, --no-contains, --merged, --no-merged, --points-at, -v, --show-current, --format, --sort), it only reads, as `git tag` list mode already does.
