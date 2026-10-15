# Round 2 fixer report — discussion

## Changes

- F1 (fixed as proposed): D4's rationale now lists, among what the downgrade lets through, a loom run whose Webster-Review budget is 0 (`webster_max_bounces`, or `review_max_bounces` when null, settable per run) and a darn run with `review_max_bounces` at 0, which skips Darn-Review.
  The bound now holds only for a loom or darn landing whose budget is above 0.
  Premise confirmed at `contracts/recipes/darn-recipe.yaml:60` and `contracts/recipes/loom-recipe.yaml:81-84`.
- F2 (fixed as proposed, option a): D2's marker-position bullet now moves every section-scoped marker row into the `markers_test.go` position table, each checking the marker sits inside its heading's section, and names the `{{.twill_directive}}`, `{{.decision_record_path}}` and `{{.support_log_path}}` rows.
  The bullet defines the set as every `claims_test.go` row with a `section:` key whose `must` holds a marker, so it also covers `{{.specs_dir}}` (line 321), `{{.parent_notify}}` and `{{.arbiter_command}}` (lines 1092-1093).
  The "section-scoped marker-placement rows included" clause and its reason are removed from the drop bullet.
  Option (a) was chosen over (b) because F3's fix keeps a heading anchor (`## The sandbox rule`), which (b)'s narrowed D1 would contradict.
- F3 (fixed as proposed): the same D2 bullet says each moved row anchors on markers, headings and the end of the body only, never on prose, and gives each kickstart row's check: end of body for `{{.kickstart_pack}}`, before `## The sandbox rule` for `{{.kickstart_pack_inline}}`.
  The rewording acceptance criterion now excepts a heading a kept marker-position row anchors on.

## Disputed

None.

## Deferred

None.
