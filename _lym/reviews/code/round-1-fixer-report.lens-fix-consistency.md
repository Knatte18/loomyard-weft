```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set for stale twins and sibling disagreements.

**Findings**

1. **LOW**: `internal/shuttleengine/engine.go:34`, the `OutcomeAPIError` constant doc.
   - The summary says: "past the re-sends Spec.APIErrorRetries allows".
   - This is the same stale claim F2 fixed in `doc.go`, `wait.go:8` and `wait.go:221`.
   - It was left unchanged, and it still contradicts the ModelRejected at-once end, which spends no re-sends.

2. **LOW**: `internal/shuttleengine/doc.go:76`.
   - The text reads "A died run whose strand is own-window keeps its last pane capture in `died-capture.txt` … and loses its strand".
   - It now covers only died runs.
   - `removeEndedOwnWindowStrand` also writes the capture and removes the strand for a model-rejected api-error run.
   - The paragraph's new line at 123, "That end removes the run's own-window strand", does not mention the capture.
   - The line-76 statement should name the api-error case, or point at 123.

3. **LOW**: `internal/shedarbiter/doc.go:15`.
   - It says "a run whose re-sends did not clear it ends as a hand-on with cause `api-error`".
   - F3 removed the same claim from the hand-on text in `call.go`.
   - This is a twin left as it was, and the sheddriver docs have the same shape: `sheddriver/doc.go:91` ("the error past them is a hand-back") and the `tackler.go` doc.
   - Neither names the rejected-model case, so both are incomplete rather than wrong.

4. **LOW**: `internal/shuttleengine/apierrorretry.go:1-2`, the file header.
   - It says the file re-sends turns "for a spec that sets APIErrorRetries".
   - The `retryAPIError` doc, `doc.go:123` and `apierrorretry.go:42` now say the rejected-model end applies whatever the spec's retries.
   - Line 2 does cover "or at once when the provider rejected the model", so this is only mild.

5. **LOW**: `internal/shuttleengine/apierrorretry.go:42`, "as a died run's end does".
   - This is a perishable cross-reference to another symbol's behaviour.
   - The new `wait.go:1971` doc for `finalizeAndRemoveDiedWindow` is unchanged and still correct.
   - Naming the died path here is allowed only as a descriptive pointer, which the code-quality rules discourage.

**Checked and consistent**
- No remaining caller of `removeDiedOwnWindowStrand`.
- `diedCaptureFileName` is still referenced by `wait_test.go`.
- `SettlerAPIErrorRetries` is still used in `tacklerprompt.go`, `shedarbiter/prompt.go` and `sheddriver/doc.go`, so no unused symbol is left behind.
- `call.go` still uses the `sheddriver` import through `sheddriver.CauseAPIError`.
- The "re-sends did not clear" string at `apierrorretry.go:87` belongs to the non-rejected branch and is correct.
- `docs/overview.md` already states the at-once end.
- The log prefix rename from "died run" to "ended run" has no test dependent on it.

**Tool calls**
1. A grep over `*.go` and `*.md` for "re-sends", "died run", `removeDiedOwnWindowStrand` and "died-capture", plus `sed` of `engine.go:30-38`. My context lacked any view of stale twins outside the edited files.
