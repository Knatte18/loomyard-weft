# Run profiles as one ladder: speed, light, solid, wide

`run-profiles-ladder` · Task · enhancement, config, loom, burler · running · Code-Bouncer

## Seen (2026-10-23)

The hub's run profiles, listed by `lyx board profiles`, grew one at a time and their names say little: `budget`, `light`, `docs` and `darn-wide`.
`budget` still sets `merge_max_bounces`, a key `loom-tail-review` retires.

## Operator proposal (2026-10-23)

One ladder of profiles, the same names for darn and loom, each setting the review rounds per perch; Code is 1 in every loom profile.

| Profile | Discussion | Plan | Code | Darn |
|---|---|---|---|---|
| speed | 0 | 0 | 1 | 0 |
| light | 1 | 1 | 1 | 1 |
| solid | 2 | 2 | 1 | 2 |
| wide | 3 | 2 | 1 | 3 |

A task needing anything else names its rounds per row by hand.

## Operator decisions (2026-10-23)

- Each profile also sets each perch's hard cap, such as its rounds plus one, so `lyx loom circling continue` can grant one more round.
- The third rung is named `solid`, not `standard`.
  A hub names its default profile per recipe instead: loom's default is `solid`, darn's is `light`.
- `docs` stays, as a darn profile only, for pure prose and config value edits: a Sonnet writer at medium effort and one Code perch round.
- The `speed` rung replaces the note `speed-loom-profile` (a loom that keeps Discussion, Plan and Webster but reviews only the end product), removed into this note.

## Evidence

`docs/research/token-audit-2026-10-22`: review loops are about a third of task spend, and Discussion and Plan review about $8 of a loom run's roughly $23 fixed cost.

## Open

- Whether a perch round count of 0, as in `speed`, turns a perch off through profile settings alone, or needs the row entry gate `gate-runs-once` builds.
- Since `code-review-one-perch` landed (2026-10-24), darn's perch is the Code-Review perch and darn reads loom's `code_*` keys (`code_max_bounces`, `code_hard_max_bounces`); `darn.review_max_bounces` and the other `darn_*`/`review_*` perch keys are retired and moved by `configcli`'s key moves.
  So the Code and Darn columns are one key: a profile cannot set Code 1 for loom and 0-3 for darn unless the key can differ per recipe, or the ladder gives up one of the two columns.
  The hub's profiles still carry the retired `darn.review_*` keys, which load through the key moves.

## Operator decision (2026-10-24): one block per recipe in each profile

- Each profile keeps one name for both recipes and holds a settings block per recipe: one for loom, one for darn.
- Batten's Seed-Child writes only the block of the recipe the task runs, so the shared `code_*` keys take the loom value (Code 1) in a loom run and the Darn column's value in a darn run.
- This keeps both columns of the ladder table without per-recipe key names; the hub's profiles are rewritten to the ladder in this form, with no retired `darn.review_*` keys left.
