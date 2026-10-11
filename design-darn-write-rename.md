# Darn: rename the writer row to Darn-Write, and close the perch gaps

`darn-write-rename` · Task · enhancement, loom, shed

## Seen (operator, 2026-10-11)

`darn` names both the recipe and its writer row `Darn` (engine `DarnWrite`), which is confusing.
The loom recipe avoids this: its rows are named for what they do (`Discussion-Write`, `Plan-Write`, `Webster`), and no row is named `Loom`.

## Wanted

Rename the darn writer row from `Darn` to `Darn-Write`, matching the loom `-Write` rows and the engine name `DarnWrite`.
Row names are durable on-disk identities pinned against `internal/loomshed` `Name*` constants, so rename the constant with the row, and keep an in-flight darn run whose status names the old row resumable.
Update every reference: recipe edges (`PR-Gate`'s `on_stuck`, the perch rows `darn-perch` added), docs, stencils and tests.

## Gaps left by `darn-perch` (PR #540 review, 2026-10-11)

- **Stale PR description.** The darn writer writes `_lyx/landing/summary.md` before the perch, and `Darn-Burler` (`fix-scope: source`) can change the code after it, so Publish can open the PR with a description of the pre-fix diff.
  Loom avoids this with a `Describe` row after `Webster-Bouncer` that writes the description from the final diff.
  Give darn the same: `Darn-Bouncer`'s `on_done` goes to a `Describe` row, then Publish; with `review_max_bounces: 0` the writer's own description stands and the run goes to Publish as today.
  A PR-Gate rejection bounces to the writer row, so the rework path also passes the perch and `Describe` before Publish; check that a re-run Publish updates the open PR's description rather than keeping the first one, in darn and in loom (whose rejections walk `PR-Rework` through `Describe` to Publish).
- **No hardcoded verify budget (operator).** `Darn-Burler`'s verify gate has `attempts: 3` hardcoded in `darn-recipe.yaml`; stamp it from `darn.yaml`'s `verify_attempts` as the `Darn` row's gate is.
- **Exhausted budget untested.** No test shows what a darn run does when the perch exhausts `review_max_bounces`, routed through loom's escalation with no decision record; add one, and fix whatever it shows.
