# Fixer-report, code review round 1

All seven findings are fixed. `lym gate test` over `./internal/seatengine`, `./internal/preflightshed`, `./internal/shedrecipe` and `./internal/loomcli` is green.

## Changes

- **F1** (fixed as proposed): `internal/loomcli/wiring_darn_test.go` gains a subtest that wires a darn seed, appends an unfilled marker to the hub's `darn-template-write` stencil, and asserts `c.env.PreflightStencils()` refuses naming seat `darn`, the stencil and the marker.
- **F2** (fixed as proposed): `Table.fills` in `internal/seatengine/prompt.go` counts a value only when the seat's value, else the table's, is non-blank. The Optional and reserved exemptions are unchanged. The `CheckMarkers` doc says "non-blank". `TestTable_CheckMarkers` gains a blank-table-value row and a blank-seat-value-over-a-filled-table-value row.
- **F3** (fixed as proposed, then widened after the self-check):
  - The `PreflightStencils` closure in `internal/loomcli/darnwiring.go` wraps a `darnTable` error as `build the darn writer table: ...; way forward: fix the config or model spec it names, then re-step`.
  - A wiring subtest with an unresolvable writer alias in darn.yaml pins the wrap.
  - Self-check fork findings showed the Preflight reason still opened with "seat stencils cannot be filled", so `stencilsReasonPrefix` in `internal/preflightshed/preflight.go` now reads "seat stencils check failed: ".
  - The log line, the `Checks.Stencils` doc and the F4 test literal follow.
- **F4** (fixed as proposed): a subtest of `TestSimpleEntries_HappyPath` in `internal/shedrecipe/entries_simple_test.go` sets `Env.PreflightStencils` to a failing func, calls the producer from `preflightEntry`, and asserts Stuck with the stencils reason.
- **F5** (fixed as proposed): `TestComposeSeatPrompt_RefusesAMarkerAbsentFromTheValueMapNamingTheSeatRole` is a table with an advisor row and a lead row expecting seat `multi`.
- **F6** (fixed as proposed, one row adjusted):
  - `TestTable_CheckMarkers` gains rows for a marker in both stencil and block (listed once) and for two unfilled seats (only the lead named, via a new `wantAbsent` field).
  - The parse-error row uses a malformed included block, not a malformed stencil body: `stencil.IncludeNames` parses the body first, so a malformed body never reaches the `TopLevelMarkers` return the row has to pin.
  - It asserts the exact `seatengine: seat "multi": stencil "seat-test-malformed": parse template` prefix.
- **F7** (fixed as proposed): the `internal/preflightshed/preflight.go` header names `NewPreflight` and `NewPreflightWithChecks`.
- Self-check follow-up: the stale `NewPreflight` mention in the `internal/loomcli/cli.go` `cwd` comment now names `NewPreflightWithChecks`.
- Gate repair: that `cwd` comment was first written with fixed-column wrapping, so the gate ran no test; it is now broken at sentence boundaries, one sentence per line.

Commits: one per finding, with F1 and F3 sharing the `internal/loomcli` files and F2 and F6 sharing `table_test.go`, so each pair is one commit. The self-check follow-ups are two further commits.

## Self-check

- fix-code-correctness, LOW: the parse-error row could pass with the `TopLevelMarkers` error return deleted — fixed with my own change (malformed block, exact prefix).
- fix-code-correctness, LOW: F3's wrap left the Preflight reason prefix "cannot be filled" on a table-build fault — fixed with my own change (prefix reworded).
- fix-consistency, LOW: `internal/loomcli/cli.go` comment still named `NewPreflight` — fixed (comment updated).
- fix-consistency, LOW: same prefix residual as above — fixed with the same change; the `Checks.Stencils` doc now also names a table that cannot be built.
- fix-proposal, LOW: same prefix residual — fixed with the same change.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
