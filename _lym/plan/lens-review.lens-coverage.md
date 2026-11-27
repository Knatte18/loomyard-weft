```yaml
lens: coverage
outcome: reported
```

Coverage lens: whether every decision, acceptance criterion and new branch has a card and a planned row that would fail if it broke. Seven findings below; nothing in the plan carries a design defect beyond these.

**Findings**

1. **BLOCKING — D8 and card 10 `Flow.Place` step 4: a window with a home waits for its parent anyway.**
   - D8 applies the parent wait and anchor only to "a window without one", meaning one without a home desktop that still exists. A homed window "goes back there".
   - Card 10 step 4 runs `awaitParent` "unless the window is the prime's". A homed window whose parent entry is missing or unsettled would wait up to 180 s, and would delay its `window_command` by the same time, before going home.
   - No `TestFlow_Place` row has a homed window skip the wait.
   - Evidence: decision-record.md D8 lines 157–163; card 10 Intent step 4; `plan:internal/kwinplace#Flow.awaitParent`.

2. **MEDIUM — D8 and D7 operand, card 11 `placeWindowCmd`: a beam's parent may be passed as `""`.**
   - Card 10's `Request` takes parent `""` to mean the window is the prime's own.
   - Card 11 resolves the parent through `hubgeom.ParentOf`, "whose `""` names the prime", but never says it maps that `""` to the prime's worktree name before building the `Request`.
   - Passed through as is, a beam's window, or a task's under the prime, is taken for the prime's own and anchors on the current desktop, against D8 ("the prime for a beam or a task under the prime").
   - No row reaches this: card 11's tier1 test asserts only the runproc record, the marker and the settled entry.
   - Evidence: card 10 `Request` paragraph; card 11 RunE bullet "resolves the parent"; `internal/hubgeom/parentof.go:34-35`.

3. **MEDIUM — D7 fallback, card 11: `""` falling to `desktop.window_placement` is never tested.**
   - The fallback from a `""` per-task lookup to the machine key lives only in card 11's RunE.
   - Card 1 tests that the lookup returns `""`. Card 11's tier1 test asserts nothing about the mode.
   - Separately, card 1's row "the task's local file wins over its worktree file" compares two layers of one worktree. The criterion "a task `run_settings` value wins over its beam's" asks for the task's local value over the parent beam's worktree. No row pins exactly that.
   - Evidence: decision-record.md acceptance D7; card 1 test bullets; card 11 test paragraph.

4. **MEDIUM — D6 clamp criterion: no row checks the clamped geometry.**
   - The criterion asks that, on a 2-row grid, each bottom-row window under a panel is released with its frame ending at the usable area's bottom. On a 1-row grid every window must be clamped and untiled.
   - That behaviour lives only in the JS that `ApplyScript` generates. `TestApplyScript_AddressesWindowsByInternalID` asserts only the internal ids, the desktops and `sendClientToScreen`.
   - The `neve-apply` recording is only parsed. No row asserts a released window's frame or zone.
   - Only the next-placement half, a clamped window counting in its zone, has a row (card 9 `TestPlan`).
   - Evidence: decision-record.md acceptance D6 bullet 2; card 5 tests; card 9 rows.

5. **MEDIUM — D12 and card 10/11: two exit paths have no row.**
   - No `TestFlow_Place` row asserts that `afterPlace` runs after a successful Apply and RecordReport. That path is D12's first listed exit; the rows cover only timeouts, `off` and a failed probe.
   - No row covers a SIGTERM arriving while `afterPlace` is running, i.e. a cancel during `afterPlace` reaching `AfterPlacement`'s context. Card 3's `CancelEndsTheCommandAndItsGroup` tests the runner alone, and card 11's wiring of a SIGTERM-cancelled context into `afterPlace` has no test.
   - Evidence: decision-record.md acceptance D12 ("runs `window_command` with no sleep after placement … a child ended by SIGTERM runs none and ends one already running"); card 10 rows; card 11 test.

6. **MEDIUM — D8: a home desktop that no longer exists has no row.**
   - Card 9 uses the home "when that desktop still exists, else the anchor".
   - No `TestPlan` row has a home desktop missing from the snapshot. "A home wins over the mode" covers only a home that exists.
   - Evidence: card 9 Intent bullet 4 and `TestPlan` rows.

7. **MEDIUM — D1 and card 6 `OpenWindow`: the failed-checks branch is never reached by a test.**
   - The criterion asks that with a check failing, "the Konsole command line is today's and no KWin call is made".
   - `OpenWindow`'s `false` branch, which calls `OpenAttach` and not `OpenAttachSeparate`, has no row. Card 6 defers to cards 12 and 13, and both stub `OpenWindow` at their seams.
   - So `TestPreLaunch` returning an error is never shown to pick the old command line.
   - Evidence: card 6 last test paragraph; cards 12 and 13 seams.

**LOW — branches with no row, and one near-operand**

- **Card 7 `Runner.run`:**
  - A run that cannot take `kwinscript.lock` within 30 s, or within the time left of `within`, fails. No row reaches it.
  - The `within` cap (D3: "within D10's window wait for at most what remains of that wait") has no row either.
- **Card 10 window wait, near-operand:**
  - D10 shares a report "that ended after its previous query, or its wait, began". Card 10 passes "that end", the end of the previous query, as `after`.
  - The cutoff for the first query (its wait's start) is not stated.
- **Card 10, missing rows:**
  - A failed `Apply` settles the entry and still runs `afterPlace`.
  - An empty plan target makes no `Apply` call.
  - The parent wait drops back to 1 s when the places file changes.
- **Card 9 ordering (D6), missing rows:**
  - Untiled windows go last in stacking order.
  - A tie in order key keeps the current order.
  - The placed window goes last when the target is not its home.
  - A root tile with no children holds no zones.
- **Card 5 `TilingOutput` (D4):** no row has a disabled output with a lower priority value, and none reaches the "no enabled output" error. Both are possible only through recordings, so the plan should say it accepts that.
- **Card 8:**
  - An unreadable or undecodable places file reading as empty has no row; only the absent file does.
  - `Settle` with no snapshot (`seenOn ""`) keeping the last desktop has no row.
- **Card 11:**
  - The prime's child is not registered in runproc; no row.
  - A failed resolution warns and still runs `afterPlace`; no row.
- **Card 12:** a directory in no worktree, with the checks passing, hands nothing off and logs at Debug; no row.
- **Card 3:** a desktop config that fails to load in `AfterPlacement` has no row.

No further tool calls were made; the context held every file this lens needed.
