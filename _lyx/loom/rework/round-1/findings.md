## Findings

### F1 — BLOCKING — template review models per segment not applied

The parent decision "Template review models per segment" (decision record, "Added after Discussion (parent, 2026-10-08)") requires `internal/loomengine/template.yaml` to set `review: opus[medium]` and `webster_review: sonnet[high]`, with every other per-segment key left empty.
The landed template keeps `review: sonnet[medium]` and `webster_review: ""` (`internal/loomengine/template.yaml`, the `review:` and `webster_review:` lines).
No plan card picked the decision up.

Fix: set `review: opus[medium]` and `webster_review: sonnet[high]` in `internal/loomengine/template.yaml`, update any test that pins the template defaults, and leave `discussion_review`, `discussion_fix`, `plan_review`, `plan_fix` and `webster_fix` empty.
