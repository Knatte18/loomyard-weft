```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. Per change, the set found and whether it is covered.**

1. **`operatorInstructions` rebuilt on `runTurnStarts` and `positionedTurnStarts`.** Covered, with the LOW finding below.
   - The only production caller is `carryover.go:161`. The only tests that reach it are in `carryover_test.go`, through `instructionEngine`.
   - The diff adds `Raw` to `instructionEngine`'s signals, which the new Raw lookup needs.
   - Other `SessionSignalParser` fakes (`readEngine`, `signalActivityEngine`, `startSendEngine`, `dispatchEngine`, `clearableBoxEngine`, `slashPasteEngine`) do not feed the carry-over path. I confirmed this only by grep: `carryover_test.go` is the only test file that names `operatorInstructions` or `CarriedInstructions`.
   - The real `claudeengine` sets `Raw` on every signal (`signals.go`, `signal.Raw = append(...)`).

2. **`turnStartOffsets` now delegates to `positionedTurnStarts`.** Covered. Its other callers at `wait.go:867`, `wait.go:933` and `wait.go:937` get the same offsets as before. The `Raw` lookup, `searchFrom` handling and offset arithmetic are unchanged.

3. **New `operatorStartedTurn` case in the held branch (`wait.go:319`).** Covered.
   - Docs: `doc.go` replaces the "cannot tell" paragraph, and `hold.go`'s file header gains the operator-turn line.
   - The orch stencil `contracts/stencils/orch/orch-template-role.md` (lines 246–251) describes only the notice's content, so it is not stale.
   - Search found no other "cannot tell" claim about the hold notice, and no PATTERN entry whose text this change breaks.
   - `PATTERN-shuttle-provider-seam`: the reading goes through the optional `SessionSignalParser`, with no engine-specific code outside `internal/shuttleengine`.
   - `PATTERN-tolerant-records`: no record format changes.
   - The fake engine's `START:<text>` doc comment is updated in `fakes_test.go`.

4. **Commit messages.** There is no `Pattern-Departure:` trailer, and none is needed. The messages match the diff, and no `PATTERN.md` change was declared.

**Findings**

- **LOW, consistency.** `internal/shuttleengine/carryover.go:67-92`. This is a behavior change from the old `operatorInstructions`, which read signals in one pass with no `Raw` lookup.
  - `positionedTurnStarts` (`wait.go:972-975`) silently drops any turn start whose `Raw` is empty or not found in the data.
  - A dropped prompt turn start now shifts which later start counts as the prompt. A dropped operator start is no longer carried over. The decision record's D1 ("reads the same instructions as before") assumed neither could happen.
  - The only trace of this change in the diff is the `Raw` that `carryover_test.go:29` now sets on `instructionEngine`'s signals. It is not documented, and `runTurnStarts`'s doc comment does not state the requirement.
  - Fix: state in the `runTurnStarts` doc comment that a turn start without a locatable `Raw` is skipped, or remove the dependency. Real engines set `Raw`, so this is not live.

**Tool calls (2)**
1. A grep for `ParseSessionSignals(` in test files and for "cannot tell", "operator's typed" and "hold notice" across `.md`, `.go` and `.tmpl` files. My context lacked the other fakes and any docs outside `internal/shuttleengine`.
2. A read of `orch-template-role.md` lines 240–256 and a grep of the shuttleengine tests for `operatorInstructions`, `carryOver` and `CarriedInstructions`. This was to check the stencil's hold-notice wording and which tests exercise the carry-over.
