---
round: 2
focus:
  - "internal/loomcli/darnwiring.go, the PreflightStencils closure in wireDarn: does a test through the wired env refuse a stencil carrying a top-level marker no darn table value fills, naming seat, stencil and marker, so that a closure returning nil would fail it?"
  - "internal/loomcli/darnwiring.go, same closure: does an error from darnTable read as a table-build failure with a fix-and-re-step way forward, distinct from the unfillable-stencils reason that only a CheckMarkers failure produces, and is that covered?"
  - "internal/seatengine/prompt.go Table.fills and CheckMarkers: does a blank or whitespace-only non-Optional table or seat value count as unfilled, consistent with stencil FillWith, with Optional and reserved markers still exempt?"
  - "internal/shedrecipe/entries_simple.go preflightEntry: is Env.PreflightStencils forwarded into the Preflight checks, and does a test fail if the forwarding is dropped?"
  - "internal/seatengine prompt tests and table_test.go TestTable_CheckMarkers: is the empty-name lead's role-named error pinned at compose time, and do rows cover a stencil parse error, a marker used in both stencil and block listed once, and first-failing-seat ordering?"
  - "internal/preflightshed/preflight.go first line: does the file header name only symbols that exist (NewPreflight, NewPreflightWithChecks)?"
---

Focus restates the sites the round 1 fixes touched; each entry is a question to settle against the diff, not a verdict.
