# Default darn's Code-Review reviewer to opus[high] in every run profile

Fixes #130.

A darn run has no Discussion or Plan review, so its Code-Review perch is the only in-run review.
That reviewer has to judge whether the writer did what the entry asks: its work items, decisions and acceptance steps, not only whether the diff is correct.
With `loom.code_reviewer_model` at its `sonnet[medium]` default, a darn reviewer has passed work the parent's PR review then found incomplete,
and hubs have worked around it per task through `run_settings`.

## What changes

- `internal/runprofile/template.yaml`: every darn block, in `speed`, `light`, `solid`, `wide` and `docs`, sets `loom.code_reviewer_model: "opus[high]"`.
  A darn task takes its profile's darn block, or the `light` default when it names none, so every darn run on a template-seeded hub gets the Opus reviewer.
  A task's own `run_settings` still layer on top and can set another model.
- The tapestry, motif, pawl, sampler, raddle and arras blocks are unchanged.
  Tapestry's Code-Review keeps the hub's model, since its Discussion and Plan reviews already check the scope.
- `internal/runprofile/doc.go` and the template's header comment state the darn reviewer default and why.

## Design decisions

- The default lives in the profiles, not in loom's template: `code_reviewer_model` is one loom key that every recipe's Code-Review perch reads, and the profile block is the per-recipe layer.
  A hub whose operator removed darn's default profile runs darn on the hub's own `loom.code_reviewer_model`.
- `docs` gets the Opus reviewer too: its writer stays `sonnet[medium]` and its single round stays solo, but the review still has to check the entry's scope.
- The sampler block no longer equals the darn block's `loom` values.
  `TestTemplateLadder` now pins that sampler carries darn's rounds, and that only darn sets `code_reviewer_model`.

## Rollout on existing hubs

The profiles reconcile carries the new key into a hub's `profiles.yaml` wherever the darn block still matches its template base.
A darn block the operator tuned gains the key as well, since the operator never set it; a value the operator set for it is kept and reported under `differs`.

## Tests

- `internal/runprofile`: `TestTemplateLadder` asserts `code_reviewer_model` is `opus[high]` in each ladder profile's darn block and absent from the tapestry, pawl and sampler blocks; `TestTemplateDocsProfile` includes the key.
- `internal/configsync`, `internal/hubreconcile` and `internal/configcli` pin the template's darn lines by text in their reconcile fixtures; those constants now carry the new key.
