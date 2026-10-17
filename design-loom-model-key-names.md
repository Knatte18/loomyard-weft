# Clearer loom model key names, one per step

`loom-model-key-names` · Task · enhancement, config, loom

## Ask (operator, 2026-10-14)

The loom config's model keys say little about what they set: `fix` reads like a flag, not "the model of the fixer in a review round".
The operator wants clearer names, and a model per step rather than one shared value for all.

## Today (`internal/loomengine/template.yaml`)

- `review` and `fix` set the reviewer and fixer model of every review segment; each takes one spec or a per-round list.
- `discussion_review`/`discussion_fix`, `plan_review`/`plan_fix` and `webster_review`/`webster_fix` override them per segment; an empty value falls back to the shared key.
- The Merge segment has no keys of its own, so it always uses the shared `review` and `fix`.
- More role models sit beside them with the same naming style: `judge` and `seed` with per-segment `*_judge`/`*_seed`, `fan_review`, `rework`, `friction`, `driver`, `arbiter`, `tackler`, `alignment`.
- "review" also appears in non-model keys (`review_max_bounces`, `review_timeout_min`, `skip_merge_review`), which adds to the confusion.

## Idea

- Name each key after the role it sets, e.g. `reviewer_model`/`fixer_model`, `plan_reviewer_model`, so a model key reads as a model key.
- Give every review segment, Merge included, its own reviewer and fixer key, so each step is set explicitly; whether a shared fallback stays is open.
- `lyx config reconcile`/Migrate rewrites old keys to the new names.
- The shape (flat role names or a nested `models:` block per segment) is an operator choice still open.

## Operator decision 2026-10-17

- Flat role names: `<segment>_reviewer_model` and `<segment>_fixer_model` for every review segment, Discussion, Plan, Webster, Merge and Darn (Darn's in `darn.yaml`).
- The shared fallback stays, as `reviewer_model` and `fixer_model`; an empty segment key takes it.
- The per-segment fan and fix-fan keys are renamed in the same style.
- `lyx config reconcile` rewrites the old keys to the new names.
- Runs as darn, after the fan keys for the code-review segments have landed.
