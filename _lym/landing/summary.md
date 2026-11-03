# Heddle: shipped instructions as headed imperative bullets

The shipped heddle entries `lym-batching` and `lym-plan-edits-before-gate` rendered into every worktree's `CLAUDE.local.md` as walls of prose.
The operator judges that agents do not take such text seriously, so both entries in `internal/heddleengine/shipped.json` are now short imperative bullets, one instruction per bullet.

## What changes

- Each entry opens with a `## ` heading of its own: "Batch your calls" and "Make every round of edits in this order".
  The rendered file now steps from the `# LoomYard's own instructions` section heading to these headings with no level skipped, as a repository entry's `##` heading sits under `# This repository's instructions`.
- The round of edits stays a numbered list because order is the point.
  Step 4's fix sequence is now four numbered sub-steps, and step 1's second-round-of-reads rule is a sub-bullet.
- The rule against reading one file in slices over several turns was stated in both entries.
  It now appears once, in `lym-batching`; `lym-plan-edits-before-gate` keeps its other prohibitions.
- `internal/heddleengine/doc.go` describes the level-one section heading and the level-two entry headings.

## Design decisions

- Every instruction in the old text is kept with its meaning unchanged, including the override of the Read tool's advice to read only a part, the reread after a command rewrites files, and the four steps of a round.
  Only the wording was split and rephrased to fit one instruction per bullet.
- `ShippedEntriesArePinned` in `internal/heddleengine/store_test.go` used to pin the whole batching sentence.
  It now pins each entry's opening heading line, so the test guards the no-heading-skip property without breaking on every wording edit.

## To check by hand

- Read the rendered `CLAUDE.local.md` of a fresh worktree and confirm the two entries read as intended and the heading levels descend without a skip.
- Compare the new bullets with the old prose for any instruction whose meaning shifted; the rewrite was not checked against an agent's behavior.
