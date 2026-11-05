# Template changes reach seed-only hub config, and light gets the lens fan back

`profiles-follow-template` · Task · bug, profiles, config, reconcile

## Seen (operator, 2026-11-05)

Commit `6d92bff1` (2026-10-28, the review-setup audit the operator accepted) set `loom.code_reviewer_fan: ""` in the `profiles` template's `light`, `docs` and `speed` blocks, so those run the Code-Review round solo.
The `profiles` module is seed-only: the hub reconcile seeds an absent `profiles.yaml` once and never fills or prunes a present one.

- The Models hub seeded its file after that commit and runs solo reviews in light, as the template says.
- The loomyard hub seeded its file before it, never got the key, and still runs light darn reviews with the lens fan.

Nothing told either hub's operator that the template had changed.
The operator's rule: lym's template is what a hub follows, and a template change must reach every hub correctly from now on.

## Template change (operator, 2026-11-05)

The operator wants the lens fan back in `light`: remove `loom.code_reviewer_fan: ""` from every recipe block of the template's `light` profile, so a light run's first Code-Review round takes the hub's fan.
`docs` and `speed` keep their solo round.
The Models hub's orch removes the key from that hub's `light` by hand meanwhile; the reconcile below then finds hub and template equal there.

## Wanted

A template change reaches a hub's seed-only config files on the next reconcile, while a value the operator set by hand stays theirs.

1. Reconcile keeps, beside each seed-only hub file, the template values it last wrote or saw (the base), committed on Bolt with the file.
2. For each key on reconcile:
   - hub value equals the base and the template moved: take the new template value, and report it under `updated`;
   - hub value differs from the base: the operator set it; keep it, and report it under `differs` with both values when the template differs too;
   - a key the template added that the hub lacks: add it, as for any other module.
   A key the template removed is removed when the hub value equals the base, else reported.
3. A hub file with no base yet (every hub today) takes as base the template as it stood when the file was seeded where that can be told, else treats a key the hub block lacks as never chosen (added) and a differing value as operator-set (kept and reported).
   On the loomyard hub this adds `code_reviewer_fan: ""` to `docs` and `speed`'s blocks and leaves `light` with the fan.
4. This covers `profiles.yaml` and every other module the hub reconcile treats as seed-only; the writer lists them in the change description.
5. The reconcile that a binary change, a start verb or a clone runs applies these updates, so no hub needs a hand step; `lym config reconcile --layer default` shows them in its dry-run.

`internal/runprofile/doc.go` and the reconcile docs describe the new rule in place of "never fills or prunes a present one".
