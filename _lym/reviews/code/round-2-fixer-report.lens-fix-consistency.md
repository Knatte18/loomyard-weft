```yaml
lens: fix-consistency
outcome: reported
```

Reviewed: the F1, F2 and F3 change set, for twins left unchanged. No list was asked for.

**Findings**

1. **MEDIUM** — `internal/gatecli/cardfixer_test.go` (new `CodexKitRendersTheCodexPrompt`, `fixerRig.engine`)
   - **Summary:** F1 item (5) is only half covered. The review's premise is that `newCardFixer` derives `engine: rig.engines.EngineName(model)` (`internal/gatecli/cardfixer.go:199`) and no test exercises that derivation.
   - **Evidence:** The new subtest sets the seam's kit engine to `"codex"` and checks the consumer, `detachfixer.go:259-277`. The kit is still hand-built in `fixerRig.seams()`, so `newCardFixer` is never called. If line 199 passed a literal or the wrong choice, every test would still pass. `detachfront_integration_test.go` has no engine assertion either. So the twin the fix proposed, asserting the kit's engine equals `rig.engines.EngineName(model)` for a choice naming another engine, was left undone.
   - **Disposition:** The fixer-report must record this as partial, or as a fix with its own change. The summary and code say it is fixed.

2. **LOW** — `internal/shedarbiter/call_test.go` (`wantEngines`) and `internal/sheddriver/tackler_test.go` (`wantEngines`)
   - **Summary:** The same helper is declared twice in two packages. A change to the engine-default logic needs both edited.
   - **Evidence:** This is an unavoidable twin, because the fakes are separate types and `sheddriver` cannot import `shedarbiter` in a test. No action is needed beyond noting it.

**Checked and in agreement**
- **F3:** no `tacklerPrompt()` remains anywhere under `internal/`.
- **F2:** the new summary bullet names the same four adapters as the review's fix. Other summary lines (20-31, 40) are generic and still true.
- **Fake `EngineName` methods:** the only other test fake with an `EngineName` signature in `*_test.go` is `tidyShuttleFake`, which gained the `engine` field. No sibling fake was missed.
- **Attach path:** `attachAndConclude` and the tackler `attach` are pinned by the new rows (shedarbiter `Rerun`, tackler paused re-attach), matching the production sites at `call.go:174` and `tackler.go:191`.

**Further tool calls (2)**
- A grep for stale `tacklerPrompt()` and other `EngineName` test fakes across `internal/`, plus `gatecli/doc.go`: my context lacked a repo-wide search for twins.
- A grep of `summary.md` and `detachfront_integration_test.go`: my context held only `summary.md` lines 45-64, and the integration test was not read.
