# Run profiles as one ladder: speed, light, standard, wide

`run-profiles-ladder` · Note · enhancement, config, loom

## Seen (2026-10-23)

The hub's run profiles, listed by `lyx board profiles`, grew one at a time and their names say little: `budget`, `light`, `docs` and `darn-wide`.
`budget` still sets `merge_max_bounces`, a key `loom-tail-review` retires.

## Operator proposal (2026-10-23)

One ladder of profiles, the same names for darn and loom, each setting the review rounds per perch; Code is 1 in every loom profile.

| Profile | Discussion | Plan | Code | Darn |
|---|---|---|---|---|
| speed | 0 | 0 | 1 | 0 |
| light | 1 | 1 | 1 | 1 |
| standard | 2 | 2 | 1 | 2 |
| wide | 3 | 2 | 1 | 3 |

A task needing anything else names its rounds per row by hand.

## Open (orch)

- Hard caps: each profile also sets each perch's hard cap, such as its rounds plus one, so `lyx loom circling continue` can grant one more round.
- `standard` equals the hub default, so the hub's own config should match it: today darn's `review_max_bounces` is 1 there, not 2.
- `docs` also sets darn's writer to Sonnet; it stays beside the ladder or goes.
- The Code column's key is `code_max_bounces` after `loom-tail-review`, and darn's perch may merge into Code-Review (note `code-review-one-perch`), which changes which keys a profile sets.
