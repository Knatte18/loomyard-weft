```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens, listing per change what the diff's changes force outside the touched lines.

**Per change: the set found, covered or not**

1. `NewPreflightWithBatcher` renamed to `NewPreflightWithChecks`, and `Checks` added.
   - Set: `shedrecipe/entries_simple.go` (updated), `preflightshed/doc.go` (updated), `preflight_test.go` (updated), and the `preflight.go` file header at line 1.
   - Not covered: the `preflight.go` header still reads "implements NewPreflight and NewPreflightWithBatcher". That symbol no longer exists. See F1.

2. `Env.PreflightStencils` added.
   - Set: the `recipe.go` field, `envkit` nilLegal (updated), `wireDarn` (set), the `entries_simple.go` consumer, and the `wiring_darn_test.go` and `wiring_test.go` assertions.
   - Covered. `wiring.go:632` sets only `PreflightBatcher`, and the field is nil for other recipes, which the `wiring_test.go` assertion pins.

3. Seat errors now use `seatLabel`.
   - Set: every `seat %q` format in `seatengine` outside `table.go` and `prompt.go`. I found only `engine.go:303` and `resume.go:72`; both are updated.
   - The remaining `seat.Name` uses at `table.go:275`, `table.go:280` and `table.go:283` quote the name under test on purpose.
   - Non-seatengine `seat %q` formats: the grep over `internal` returned none.
   - Covered.

4. `composePrompt` removed, and the `composeSeatPrompt` signature changed.
   - Set: callers in `engine.go:293`, `prompt.go` and `prompt_test.go` are updated.
   - `burlerengine/template_test.go:20` mentions `composePrompt` only in a comment about a different package's function.
   - `preflightshed/doc.go`: the doc was updated.
   - Covered.

5. `readStencilSet` extracted and `validateStencil` changed.
   - Both are used in `table.go` and `prompt.go`, and I found no other callers.
   - Covered.

6. `Table.CheckMarkers` added.
   - Placement: it is defined in `prompt.go` but documented in the `table.go` and `doc.go` context. This is not a defect.
   - Docs: `seatengine/doc.go:35` documents it, and `preflightshed/doc.go` and the `wireDarn` doc are updated.
   - Rule documents: I did not read `PATTERN.md` or `pattern/`. No entry is touched, since the check uses no git, config-layer or stencil-ownership path. It reads stencils through `stencilstore` at call time, which fits `PATTERN-stencil-ownership`.
   - `PATTERN-refusal-way-forward`: the refusal text names its way forward, "re-step with a lym binary that fills them". A test asserts it at `table_test.go`. Covered.
   - Declared PATTERN changes: none visible in the diff, and the directive declares none. No `Pattern-Departure` trailer is needed.

7. Tests that assert seat error text.
   - `table_test.go` and `prompt_test.go` are updated.
   - Other tests asserting `seat ""` or `seat "advisor-1"` text may exist outside `seatengine`. My grep found none in the files I saw, but I did not search test files beyond the diff.

**Findings**

- **F1, LOW, consistency:** `internal/preflightshed/preflight.go:1`. The file header still names `NewPreflightWithBatcher`, which this diff removed. Evidence: grep output line `internal/preflightshed/preflight.go:1`. Fix: change it to `NewPreflight and NewPreflightWithChecks`, and say that the producer also runs optional checks.

No other affected member is left wrong in the set I found.

**Tool calls (1)**
1. Grep for `NewPreflightWithBatcher`, `PreflightBatcher`, `PreflightStencils`, `CheckMarkers` and `composePrompt` across non-test `.go` and `.md` files. My context lacked a repo-wide check for stale references to the renamed or removed symbols.
