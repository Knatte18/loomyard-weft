# The Discussion-Review round asks the parent orch, and its chair keeps every parent finding

`discussion-review-parent-seat` · Task · feature, loom, orch, config, burler · running · Drift-Expect

Decision record: [Sketch](decision-discussion-review-parent-seat.md)

Depends on: [`parent-review-ask-verbs`](design-parent-review-ask-verbs.md)

Filed from the Sketch `parent-reviews-decision-record`.

Takes: parent-review-ask-verbs' ask record API, deadline cap, config keys and verbs; NotifyParent's routing; the assistant seat as the model for the consolidation part and the copy beside the review
Delivers: the asking rule and creel ask in Discussion-Burler-Review, signed as <shortname>:LOOM; the parent consolidation stencil and marker; Finding's parent_ids and the review format stencil's key; the parent coverage gate entry; the copy of a done report beside the review; the orch-role section for [discussion-review request]
Exports: the [discussion-review request] creel line, Finding.ParentIDs, the parent coverage gate entry, an orchcli post to a given strand signed as a program

The part changes these packages:

- internal/burlerengine (about 500 changed lines)
- internal/shedadapters (about 300 changed lines)
- contracts/stencils (about 90 changed lines)
- internal/orchcli (about 50 changed lines)
- internal/loomcli (about 40 changed lines)
- contracts/orchrole (about 40 changed lines)
