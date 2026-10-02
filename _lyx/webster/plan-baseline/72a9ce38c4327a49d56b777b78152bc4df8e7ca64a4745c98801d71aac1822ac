# Card 3 — Delivery-prompt and reviewer-brief stencils

**Create:**
- `contracts/stencils/loom/loom-template-parent-review-delivery.md`
- `contracts/stencils/loom/loom-template-parent-review-brief.md`
- `internal/loomengine/parentreview.go`
- `internal/loomengine/parentreview_test.go`
- `internal/loomengine#ParentReviewDeliveryPrompt`
- `internal/loomengine#ParentReviewBrief`

**Edit:**
- `contracts/stencils/stencils.go`

**Uses:**
- `internal/stencilstore#Read`
- `internal/stencilstore#Path`
- `contracts/stencils/burler/burler-step-2-review.md`
- `internal/loomengine#composePrompt`

**Intent:**
Add the two stencils decisions D5 and D7 render, registered as embedded defaults and read at call time per the Stencil Ownership Invariant.

`loom-template-parent-review-delivery` is the delivery prompt, rendering to one line (the shape shuttle's send requires, as `orch-template-resume` does), with markers `slug`, `request_path` and `reviewer`.
It tells the writer to send one message through Claude Code's `SendMessage` addressed to the reviewer's full name, carrying the slug, the request path and an instruction to review it with a one-shot fork; then to run `lyx loom review delivered`, or `lyx loom review delivered --failed <reason>` when the send failed; then to end its turn without changing the discussion.
Its wording follows the driver's parent notice in `internal/loomcli/driverprompt.go`.
The SendMessage wording lives in this stencil only, never in `shuttleengine` or `reedengine` (Shuttle Provider-Seam Invariant).

`loom-template-parent-review-brief` is the reviewer brief, with markers `slug`, `decision_record_path`, `support_log_path` and `review_format_path`.
It tells the reviewer to read both discussion files, to read `lyx board list` and `lyx board get` any entry touching the task, and to judge whether the design fits planned work without duplicating or contradicting it.
It points at the review-output format by the path of the deployed `burler-step-2-review` stencil and never restates it (Producer Pointer-Rule Invariant).
It states that a review with any finding is a reject and an approve carries no finding the writer must act on, and gives the exact submit lines: `lyx loom review approve <slug>` and `lyx loom review reject <slug> <review-file>`.

`stencils.go` gains the two embedded vars and two `entries` rows.
`parentreview.go` holds the two compose functions, shaped like `composePrompt`: read the stencil, fill it through `internal/stencil`, return the text; `ParentReviewBrief` fills `review_format_path` from `stencilstore.Path` for `burler-step-2-review`.
`ParentReviewDeliveryPrompt` refuses a render that contains a newline.

`parentreview_test.go` renders both against a temp stencils directory seeded from the embedded defaults and asserts the delivery prompt is one line naming the reviewer, slug and request path, and the brief names both discussion paths, the format path and both submit lines.

**ImpactSummary:** Two new registered stencils and their compose functions; the registry's on-disk tree test covers the new entries, and no existing stencil changes.
