```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per-behaviour list**

| Behaviour | Pinning test |
|---|---|
| Validate errors name the seat by role, lead as the prefix | `TestTable_Validate` rows (`seat "multi": has no output`, `seat "multi-advisor-1": …`), `table_test.go` diff |
| `composeSeatPrompt` fill error names the role | `TestComposeSeatPrompt_RefusesAMarkerAbsentFromTheValueMapNamingTheSeatRole`, but only for an advisor (`multi-advisor-1`), not the empty-name lead |
| Lead label in the `startSeat` start error (`engine.go:303`), the `probeSeat` error (`resume.go:72`), and the `strandNames`, `seatValues` and `readStencilSet` errors through compose | none |
| `CheckMarkers` reports a stencil marker | `TestTable_CheckMarkers` row "a stencil marker and a block marker with no value" |
| `CheckMarkers` reports a block marker, with sorted output and the way forward | same row (`focus, topic`, "way forward: re-step…") |
| A table value, seat value, Optional name or reserved marker fills a marker | rows 1 and 2; `{{.seat_name}}` in the lead stencil covers reserved |
| A value on another seat does not fill | row "a value on another seat" |
| An advisor seat is checked, and the seat is named | row "an advisor stencil marker with no value" |
| An unreadable stencil is returned | row "an unreadable stencil" |
| Stencil parse failure in `CheckMarkers` (`TopLevelMarkers` error, `prompt.go:292-294`) | none |
| Dedup of a marker used in both stencil and block, and first-failing-seat ordering across several unfilled seats | none |
| Present-but-empty non-Optional value | none; `CheckMarkers` counts it filled (`fills`), while `FillWith` counts it unfilled (`unfilledTopLevelMarkers`) |
| Preflight stencils check: Stuck reason, logging, cancellation, pass, batcher-before-stencils order | `TestPreflight_StencilsCheck` |
| `wireDarn` sets `PreflightStencils` for a darn seed and nil for a tapestry seed | `wiring_darn_test.go:116`, `wiring_test.go:183` |
| `PreflightStencils` actually refuses an unfilled marker (calls `CheckMarkers` with the stencils dir and project directive) | none; the wiring test only asserts nil on the shipped stencils |
| `preflightEntry` passes `env.PreflightStencils` into `Checks.Stencils` (`entries_simple.go:25`) | none visible in the diff |
| `PreflightStencils` nil-legal in envkit | the envkit self-check (no diff change needed) |

**Findings**

1. MEDIUM, design. Location: `internal/loomcli/wiring_darn_test.go:116-120`. Summary: the wiring test checks only that `PreflightStencils` is non-nil and passes on the shipped stencils. Replacing its body with `return nil`, or dropping `CheckMarkers`, would keep it green. Evidence: `darnwiring.go:261-267`; no test seeds a stencil with an extra marker and expects the refusal.

2. LOW, design. Location: `internal/shedrecipe/entries_simple.go:25`. Summary: nothing pins that `preflightEntry` forwards `env.PreflightStencils` to `Checks.Stencils`. Dropping it would silently disable the check in every real run. Evidence: the diff has no `shedrecipe` test change.

3. LOW, design. Location: `internal/seatengine/engine.go:303`, `resume.go:72`, `prompt.go:27,53-90`. Summary: the role labels in the start, probe, `strandNames` and `seatValues` errors have no test. The empty-name lead, which is the reported bug, is pinned only on the `Validate` path. Evidence: `prompt_test.go` pins only the advisor's fill error.

4. LOW, design. Location: `internal/seatengine/prompt.go:283-308`. Summary: `CheckMarkers` branches with no row: a stencil parse error, deduplication across stencil and block, and refusal of the first failing seat when several fail. Evidence: the `TestTable_CheckMarkers` rows listed above.

5. LOW, design. Location: `internal/seatengine/prompt.go:311-315`. Summary: `fills` treats a present but empty or whitespace-only non-Optional value as filled, while `FillWith` treats it as unfilled. The behaviour is unpinned, and this is a possible false negative. Evidence: `stencil.go:298` (`strings.TrimSpace(values[name]) != ""`); `TestTable_CheckMarkers` has no such row.

**Tool calls:** none. The inherited diff and sources already covered everything.
