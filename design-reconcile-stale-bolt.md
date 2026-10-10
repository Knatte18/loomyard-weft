# fabric reconcile on a stale Bolt: seed commits conflict on push, yet ok:true

`reconcile-stale-bolt` · Note · bug, fabric, config

Imported from [issue #478](https://github.com/Knatte18/loomyard/issues/478).

Seen 2026-10-09 on a clone that had not been used since 2026-10-05: its local Bolt (`<hub>/_board`) was 1829 commits behind `origin/main`.
After a freshly deployed `lyx`, `lyx fabric reconcile` in the prime reported `ok: true` while `hub_config_detail` and `warp_binding_detail` both said the push failed:

```
gitrepo: git pull --rebase: Rebasing (1/3)
error: could not apply cd6b67cf... lyx: seed stencils
```

The root pre-run had seeded stencils and specs, and reconcile had written the hub-wide `board.yaml`, as three local commits on the stale Bolt.
Upstream already had newer versions of the same files, so the rebase conflicted.
The rebase was aborted, leaving Bolt 3 ahead and 1829 behind, and every later lyx command repeated the failure.
The fix was by hand: `git reset --hard origin/main` in `_board`, then `lyx fabric reconcile` again, which was then clean with nothing to seed.

## Expected

- Seeding and the hub-config write pull Bolt before they write, so they compare against upstream instead of a stale local copy.
- A seed commit is regenerated content: when its push conflicts, lyx drops it, fast-forwards Bolt and seeds again, rather than leaving a conflicting local commit for the operator.
- `ok: true` is not reported when the hub config or warp binding could not be pushed; the failure is surfaced as a failure, with the command that resolves it.

Related: the stencil seed ping-pong issue (two prod-mode binaries overwriting each other's stencils on Bolt).
