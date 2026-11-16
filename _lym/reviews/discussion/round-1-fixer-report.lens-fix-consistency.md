```yaml
lens: fix-consistency
outcome: reported
```

Task: check, through the fix-consistency lens, that the fix's change set agrees with the sibling decisions, criteria and cross-references around it.

**Findings**

1. **MEDIUM**: D1's new producer-table scan has no carve-out for the specs other decisions keep out of parity.
   - **Changed text:** D1, "a scan fails on a production launch-spec literal in a package the table has no entry for", and the matching first acceptance criterion.
   - **Problem:** Three kinds of spec literal would fail that scan:
     - the hand-written `lym shuttle run` spec, which D4 marks hand-written with no rendered prompt to compare;
     - the orch spec, which D4 pins to Claude and Scope puts out;
     - `lym shuttle open`, which Scope puts out because it carries no role.
   - **Evidence:** The new scan in D1 names no exemption. D4 says "only `lym shuttle run`… marks its spec hand-written in place of a prompt engine". Scope's Out list names "The orch role" and "`lym shuttle open` sessions".

2. **MEDIUM**: The new D12 nudge criterion claims more than the D12 body does.
   - **Changed text:** "A Codex chain of lookup-only shell responses, a shell read of a line range, or a run of `apply_patch`-only responses gets the nudge line…"
   - **Problem:** The criterion says any line-range read gets the nudge. The D12 body says a sliced read is "judged against `slice_lines` and the read-again rule as a sliced Read is". The criterion also leaves out the `lookup_run` and `edit_run` thresholds that the body names.
   - **Evidence:** Claude's `readSliceNudge` fires only for "a file under %d lines" or "a file you already read that has not changed since" (`internal/shuttleengine/claudeengine/batchingnudge.go`, the `sliceShortReason` and `sliceReadReason` constants).

3. **LOW**: The new D5 criterion is in the wrong place.
   - **Changed text:** "`Prepare` refuses a Codex launch whose `developer_instructions` argument passes 128 KiB… (D5)".
   - **Problem:** The criteria list runs in decision order and already has a D5 line ("A Codex launch passes its standing items as `developer_instructions`…"). The new line was appended after D12's lines instead of being merged into that D5 line.

4. **LOW**: Two support-log entries still describe parity as seatengine seats only.
   - **Problem:** D1 now covers every producer of a launch spec, but these entries were not updated:
     - Turn 3: "board-scope, parity per role, not per seat: fixed, D1's test runs every seat in `seatengine`'s tables."
     - Q1: "A manifest read back from each engine's launch, per seat in `seatengine`'s tables".

5. **LOW**: D12 moves code to a new shared package that the Constraints don't mention.
   - **Changed text:** D12, "the check moving to a package both engines import".
   - **Problem:** The `PATTERN-shuttle-provider-seam` constraint lists what stays in `claudeengine` and `codexengine`, and says "the rollout reading for the batching nudge stays in `codexengine`". It doesn't name the shared package the read-only lookup check moves to, which `PATTERN-shuttle-provider-seam` would govern.
   - **Evidence:** The lookup check is `isReadOnlyLookup` in `internal/shuttleengine/claudeengine/bashlookup.go`.

6. **LOW**: The plan writer's note covers seats only.
   - **Problem:** The note says "`seatengine.Prompts` already renders every seat's prompt without a session". Producers outside `seatengine` are now in the test, and nothing tells the writer how their prompts get rendered for each engine without a session.

I made no further tool calls; my context already held everything this lens needed.
