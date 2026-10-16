# Plan burler fix, round 2: edit-test-file on a file with no members

`edit-test-file` blocked card 12, which edits `cmd/lyx/seamsignature_test.go`, a test file holding only two blank `var _` slices, so quarry lists no symbols for it.
The check asks for member glyphs under Edit, but such a file has none, so no target spelling passes, and the check's source is not in this tree to read.
The fix was to change the plan's code: card 12 now names the two slices, so they become Create targets.
A memberless test file may deserve its own allowance in the check.
