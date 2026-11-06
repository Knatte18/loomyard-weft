```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. Per change, the set found, and whether the diff left it right:**

1. **Fanned stencil and directive (new `burler-template-assist-reviewer-fanned`, `seat-directive-assistant-fanned`).**
   - Set: registry entries and `roleOpeningStencils` in `stencils.go`, `markers_test.go`, the `PATTERN-stencil-ownership` parent and edit directive rule, and the docs and stencil indexes.
   - Covered.
     - The parent and edit directives are both rendered in the new stencil.
     - Both stencils are registered.
     - The markers row is added.
     - A grep for the stencil names found no doc or index outside the changed files.
   - The step and engine-reading stencils contain no "nothing else" or "only file" write-surface wording that would contradict the fanned surface.

2. **`burler-assist-cluster-fan.md` ("one exception" wording).** Covered. No other stencil refers to that exception.

3. **`Seat.SubagentReports`, `output_files`, `clearAssistantFiles`, `Validate`.**
   - Set: `seatValues`, `seatSpec` (outputs stay report-only), `clearAssistantFiles`, `Validate`, and `CheckMarkers`.
   - Covered by the diff and its tests.
   - The comments around the changed code were left stale (findings 1 and 2 below).

4. **`lensreport.Returned` and `Path`, and the lens-report spec.**
   - Set: the spec's Readers section, `PATTERN-sole-parsers`, `PATTERN-leaf-packages`, the package doc and the other users of `lensreport`.
   - Covered.
     - The spec and PATTERN entries are updated.
     - The leaf rule "standard library only" still holds, since `bytes` and `os` are standard library.
     - The other users, `planparser` and `loomengine`, only use the naming functions.

5. **`round-<N>-usage.yaml` keys `assist_ending` and `assist_lenses_missing`.**
   - Set: `roundUsage`, `recordRoundAttempt`, `acceptedExit`, `shedadapters` `doc.go`, `PATTERN-tolerant-records`, and other readers of the record.
   - Mostly covered.
     - The usage-record decode is non-strict, so an older binary tolerates the new keys.
     - A grep found no other reader of the keys.
     - Finding 3 below is a partial gap.

6. **`AssistOutcome.MissingLenses` and the `keepAssistReports` signature.** Covered. `concludeReview` is the only production caller, and `TestKeepAssistReports` is updated for the new argument.

7. **Declared PATTERN changes.** The focus declares none. The `PATTERN-sole-parsers` addition matches the code that lands.

8. **Commit messages.** Neither commit carries a `Pattern-Departure:` trailer. The claims in the body of each commit hold against the diff.

**Findings**

1. **LOW, consistency.** `internal/seatengine/table.go:35`. The comment on `assistantDirectiveBlock` still says the block is the one "which every assistant seat's stencil includes". A fanned assistant's stencil now includes `fannedAssistantDirectiveBlock` instead (`table.go:38,140`), so the sentence is false.

2. **LOW, consistency.** `internal/seatengine/doc.go:112`. The package doc says "every assistant's seat-directive-assistant". The paragraph the diff added at `doc.go:115` says a fanned assistant includes the fanned directive in its place, so the two statements contradict each other.

3. **LOW, design.** `internal/shedadapters/roundusage.go:86-93` with `burlerreview.go:425-428`.
   - `recordRoundAttempt` resets `Assist` and `AssistFan` at a new attempt but not the new `AssistEnding` and `AssistLensesMissing`.
   - `acceptedExit` writes the two new keys only when `result.Assists` is non-empty.
   - A later accepted attempt of the same round with no assistant would keep the earlier attempt's `assist_ending` and `assist_lenses_missing` beside `assist_fan: ""`. That is a stale record of the very thing this change adds.
   - The doc on `recordRoundAttempt` calls it "the only writer of the three keys" and does not cover the two new keys.

**Tool calls (2 of 6, both in one message):**
- `grep` of the step, fork-reading, assist-reading and consolidation stencils for write-surface wording that could contradict the fanned surface. My context held only the cluster, template and directive stencils.
- `grep` for `KnownFields` in `shedadapters`, and for `assistantDirectiveBlock` and `SubagentReports` in `seatengine`. I needed to know whether the usage-record decode is strict, and whether other references to the solo block were left stale.
