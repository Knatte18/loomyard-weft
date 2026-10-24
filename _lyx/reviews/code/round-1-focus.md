---
round: 1
focus:
  - "docs/overview.md: the parent's ide-spawn edits (`ctrl+alt+a` binding) and this task's `decision add --from` paragraph sit in different bullets; check the merged file keeps both intact and the `--from` paragraph is not displaced or duplicated."
  - "internal/loomcli/decision.go and decision_test.go against the amended board entry: does one `--from` call write all entries in one commit and send one notice naming the new headings rather than full texts, and does the Merriam-only inline-text case match the docs/overview.md wording?"
  - "Failure paths of the batch write: a discussion-check finding or a refused entry mid-batch must restore the record whole, with no partial append and no commit; check internal/discussionparser/append.go and its test."
  - "Merge-In consistency: the parent changes since the settle must not have left the settled `--from` code, its docs (doc.go, overview.md) or the orch stencil line out of step."
---

Amendment round: review only what changed since the settle patch.
The overlap note names `docs/overview.md` as the one file both sides changed.
