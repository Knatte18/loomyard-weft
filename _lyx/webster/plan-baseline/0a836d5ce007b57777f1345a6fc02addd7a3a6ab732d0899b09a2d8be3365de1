# Card 3 — `loom.yaml` gains the checkpoint and the review budget

**Edit:**
- `internal/loomengine/config.go`
- `internal/loomengine/template.yaml`
- `internal/loomengine/config_test.go`
- `docs/overview.md`

**Intent:**
D3: two new `loom.yaml` keys with defaults in the embedded template, loaded through the existing strict `configengine.Load` path (`PATTERN-config-strictness`).

- `review_circling_checkpoint: 3` — the first judged round from which a review segment's judge may rule `CIRCLING`.
- `review_max_bounces: 5` — the bounce budget of every Bouncer review segment's rows (Discussion-Review, Plan-Review, Webster-Review).

Add `ReviewCirclingCheckpoint` and `ReviewMaxBounces` to `Config` with those yaml tags, and one template line each with a trailing comment in the file's existing style.
In `LoadConfig`, reject a value below 1 for either key with an error naming the key and the value, in the shape of the existing `parent_review_wait_min` check.
A checkpoint larger than the rounds the budget allows is accepted: such a run never rules `CIRCLING` and reaches the budget escalation instead.
Update `config.go`'s file header, which enumerates the validated keys, and `docs/overview.md`'s loom config line, which enumerates the `loom.yaml` keys.

Tests in `config_test.go`: a `loom.yaml` without the new keys loads checkpoint 3 and budget 5; explicit values load verbatim; zero and a negative value for each key fail with an error naming that key; a checkpoint above the budget loads.

**ImpactSummary:** Two new config fields with template defaults; no reader until Cards 4 and 9.

**Commit:** 3: loom.yaml gains review_circling_checkpoint and review_max_bounces
