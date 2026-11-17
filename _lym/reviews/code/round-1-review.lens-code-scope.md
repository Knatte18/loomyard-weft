```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the board entry's goal points and checked every goal point for a change that builds it.

**Decisions.** The answer key is a decision record alone. It has no `Added after Discussion` entries.
- **G1:** count an edit toward a loop only when it undoes or repeats an earlier one, for example the same `old_string` edited back and forth or the content returning to an earlier hash, instead of counting every edit.
- **G2:** failing G1, exempt one-document roles or give them a higher count. This is a fallback that applies only if G1 is not built.
- **G3:** the hub's shuttle config sets `edit_loop_edits: 0`. The fix sets it back to a working threshold.

**Per change:**
- **C1. Content hash on each edit record.** Files: `claudeengine/editrecord.go` (`contentHashKey`, `ContentHash` field, `fileContentHash`), `claudeengine/signals.go` (`fileEditSignal` reads the hash), `sessionsignal.go` (`SessionSignal.ContentHash`), `claudeengine/doc.go`. Traces to G1, the "content returning to an earlier hash" route.
- **C2. The fold counts only repeats.** Files: `editloop.go` (`editedContent`, `editHistory`, `fold`, `clone`, file header, `watchEditLoops` doc), `config.go` (`EditLoopEdits` doc), `shuttleengine/doc.go`. Traces to G1.
- **C3. Notice wording.** In `editloop.go`, `editLoopNotice` now says "back to content it held earlier N times". Forced by C2: the old "edited N times" text would misstate the count.
- **C4. Template default 4 → 3 and its comment.** Files: `template.yaml`, plus the `TestLoadConfig_EditLoop` row in `config_test.go`. Traces to G3, because the template's count is the working threshold a dropped hub value falls back to. It is also forced by C2, since the key now counts repeats.
- **C5. Reinterpretation.** `reinterpret.go` adds `DropEditLoopEdits` and `EditLoopRepeatsReinterpretation`, and `configreg.go` registers them on the shuttle module. Traces to G3: it drops the hub's `edit_loop_edits: 0` (`_bolt/_lym/config/shuttle.yaml:1`) so the template's 3 applies. Dropping a non-zero value too is forced by the meaning change, because PATTERN-config-layers requires a reinterpretation when a value's meaning changes (`pattern/PATTERN-config-layers.md:116`).
- **C6. `docs/overview.md`, lines 665 and 708.** Forced by C1 and C2, since the old doc text would be stale.
- **C7. Tests and fakes.** Files: `editloop_test.go`, `editrecord_test.go`, `reinterpret_test.go`, `fakes_test.go` (the `EDIT:` line gains a hash). Forced by C1, C2 and C5.
- **C8. `configreg_test.go`.** The creel-only test became the table-driven `TestModules_RegisterTheirReinterpretation`, with a shuttle row. Forced by C5; folding the shuttle check into the existing test rather than adding a new one follows the one-test-per-behavior rule.
- **C9. `editrecord.go` file header**, "the edited file" → "an edited file". A wording touch in a file C1 already rewrites. Forced by C1 and too trivial to file.

**Per decision:**
- **G1:** built by C1 and C2, with C3, C6, C7 and C8 forced by them.
- **G2:** none, which is correct. G2 applies only if G1 fails, and the diff builds G1.
- **G3:** built by C5 and C4.

**Findings:** none. Every change traces to G1 or G3 or is an edit a traced change forces, and every goal point that applies has a change that builds it. Nothing needed a `Pattern-Departure:` trailer, and the range carries none.

Tool calls: none. The inherited context already held the diff, the decision record, the hub's `shuttle.yaml` and `PATTERN-config-layers`.
