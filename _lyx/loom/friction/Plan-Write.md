# Plan-Write: splitting a test-first card from the reorder card

I first wrote two cards: one adding a same-tip archive-reuse test to `remove_archive_integration_test.go`, then the reorder card that also edits that file.
`lyx loom validate-plan` refused it with `containment-file-overlap`, because two cards may not target the same file.
The Plan prompt's card-granularity rules never mention this constraint, so the split looked valid until the gate ran, and I had to fold both cards back into one.
