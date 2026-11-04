```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: no findings. I reran the search for the sets that F1 and F3 name, and F4 needed no search.

Findings I left unchecked: none.

- **F1 and F3 (callers of the prompt delivery):** the search found four callers. The two shuttleengine callers, `wait.go:297` (via `deliverPromptOnAttach`) and `wait.go:334` (via `deliverPromptAtHold`), both reach `DeliverPendingPrompt`. The third is `internal/burlerengine/fix.go:263`, which calls `DeliverPendingPrompt` through an interface. The `clearPendingSend` fix sits inside `DeliverPendingPrompt` (`run.go`), so that caller gets it too. The fourth, `loadSkillsThenPrompt`'s first prompt send, calls `sendThroughPause` directly. It runs before any failed prompt send could have left a pending-send record, so it needs no clearing. The `run.covered` guard is in `deliverPromptOnAttach` and `deliverPromptAtHold`. The burlerengine caller has no `run.covered` guard; that path was not in the review's set and the fix did not change it.
- **F4 (error returns in `writeOpenFiles`):** the deferred `RemoveAll` is registered right after `MkdirAll` and triggers on the named `err`, so it covers every later return. `OpenLaunch` has no other file-writing path.

Tool calls: one `grep` for the callers of `DeliverPendingPrompt`, `deliverPromptOnAttach` and `deliverPromptAtHold` across non-test Go files. My context held only the shuttleengine files and lacked the other callers, burlerengine in particular.
