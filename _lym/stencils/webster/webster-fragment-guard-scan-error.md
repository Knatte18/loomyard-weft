<!-- This is the guard-scan-failure line of webster's card-gate list in a fork or recovery prompt.
     guardNotice (guardstep.go) reads it through stencilstore, fills its one marker, strips this banner and trailing newline,
     and appends the text to the card gate list when the scan for `//lyx:guard` tests failed on the content of a test file.
     guard_error is the scan error's text.
lyx-stencil: sha256=032416f9cbebfcbf17b0542f2a3ec5d232c7ec934a7da2d03483e150de4f83ca build=d4824a3d68de466ba3db5015cf6212f7d41126f7 time=2026-10-14T16:27:00Z -->

The guard scan failed with `{{.guard_error}}`, so no card gate above runs the repository's `//lyx:guard` tests.
The plan-level verify still runs them.
