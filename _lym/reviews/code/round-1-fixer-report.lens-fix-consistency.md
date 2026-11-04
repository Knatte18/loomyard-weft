```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix set for sibling or twin text left stale (lens fix-consistency).

Findings:

1. **LOW**, `internal/shuttleengine/doc.go:148-152` ("Pending prompt" section) against the new `DeliverPendingPrompt` and `deliverPromptOnAttach`.
   - **Entry delivery:** Line 151 says Wait calls `DeliverPendingPrompt` "once on entry for a run Attach reconstructed whose output files are missing". `deliverPromptOnAttach` now also returns when a pause covers the run, and the section never says so. The held-turn-end clause on the same line already states the covering-pause condition. The two occasions are now documented asymmetrically, although the code (`hold.go` `deliverPromptOnAttach`, `run.covered`) treats them alike.
   - **Pending-send record:** Line 150 does not mention that the call clears the pending-send record before sending. That is a new behaviour of `DeliverPendingPrompt` (`run.go`, the added `clearPendingSend` call). The section covering the pending-send record (`doc.go:390-396`) says the next send reads and clears the record, and the line-403 note that the conditioned send never reads it names no other exception. A reader of those lines would conclude the redelivery honours the record, which is the premise F1 removed.
   - **Suggested edit:** Add "and no pause covering the run" to the entry clause on line 151. Add "clears any pending-send record first" to line 150.

2. **LOW**, `internal/shuttleengine/claudeengine/command.go`, the `OpenLaunch` doc comment against the new `writeOpenFiles` cleanup.
   - **Stale doc:** The `OpenLaunch` doc says it "creates dir and writes into it settings.json ... and the append-system-prompt file". It does not say a failed write removes `dir`. Only the `writeOpenFiles` comment was updated. The doc also says "An engine told no board directory writes nothing, leaves dir uncreated", which stays true.
   - **Why it matters:** `Open` (`open.go`) removes `sessionDir` only when `AddStrand` fails, and now `OpenLaunch` cleans up its own failure. The two cleanups agree, but the contract an `OpenLaunch` caller reads is incomplete. Add "A failure after dir is created removes it."

3. **no finding (checked):** The `hold_test.go` header and the `attach_test.go` header and docstring still match the rows. `TestAttach_DeliversTheDeferredPrompt`'s docstring says it types "once at Wait entry when its output files are missing and no turn of the prompt started, and types nothing when every output file exists". It does not mention the covered and send-failure rows, which is acceptable but slightly incomplete.

4. **no finding (checked):** `echoReed.sendFailures` and `SendTextErr` coexist. `sendFailures` is checked first, and the api-error `sendFails` rows leave `sendFailures` at 0, so the older rows are unaffected.

Calls made (3 of 6):
- Bash grep of `docs/`, `shuttleengine/doc.go`, `claudeengine/doc.go` and `creelengine/doc.go` for deferred-prompt and pending-send text. My context had no doc files.
- Read `doc.go` lines 386-409. My context had no view of the pending-send paragraph.
- Bash grep of `doc.go` and `docs/overview.md` for `DeliverPendingPrompt` and similar. I needed the line numbers of the Pending prompt section.
- Read `doc.go` lines 145-154, the Pending prompt text itself.
