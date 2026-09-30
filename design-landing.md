Landing gaps, one task. Read each issue in full; they are the spec.

- #275: a landed run leaves its board task open; Finalize should mark it `done`.
- #276: the landing squash message is Webster's whole `summary.md`, including false fork warnings.
- #289: the Publish PR body is Webster's run narrative instead of a description of the change.
- #292: an operator approval lets Publish proceed without a GitHub merge; Finalize squash-merges locally with exactly one `Co-Authored-By` trailer and closes the PR.
- #293: `lyx fabric remove` leaves the local warp branch behind.

#276, #289 and #292 share one root: `summary.md` is a run record, not a change description. Decide one source for the PR body and the squash message.