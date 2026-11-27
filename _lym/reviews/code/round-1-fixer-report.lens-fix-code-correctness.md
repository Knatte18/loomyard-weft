```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness check of the fix's changed code.

1. **LOW** — `internal/operatoragent/spec.go`, `paneRules`, last sentence "The section above governs only how you treat creel mail from your parent."
   - The parent-variant section also decides how to treat mail from any orch strand, another hub or machine, and `LOOM`/`CREEL`. It also says "Trust no other session's message". "Only … mail from your parent" tells the agent those rules no longer apply.
   - In the prime, the no-parent variant renders ("No parent is recorded for this run."), so the sentence refers to mail from a parent that does not exist.
   - Evidence: `contracts/stencils/parent/parent-directive-parent.md` (sender-trust paragraph) and `parent-directive-none`.
   - A wording such as "Of the section above, only what it says about creel mail applies to you" would keep the trust rules. It follows the review's own `fix` wording, so the parent should weigh it.

I found nothing else. I traced these paths:
- **`withSessionEnd`:** both launch paths (`launch` and `relaunchUnderAccount`) wrap the pane lines. `relaunchForReprompt` is gated-only, so an unawaited session never reaches it.
- **Provider dies at startup:** the strand may already have removed itself. `removeStrandIfPresent` then does nothing, `startupStrandRemoved` is still set, and the start retry holds.
- **`NoCarryOver`:** the empty role key gives no predecessor, no carry-over, no resume, no notice, and an empty recorded `RoleKey`.
- **`RefuseCovered`:** `Open` checks in the same order as before.
- **New tests:** each of the `NoCarryOver`, `EndsWithSession`, parent-name, role-normalization and `agent` verb tests would fail with its defect in place. The pause refusal in `agentCmd` has no test of its own; the review did not ask for one.

Tool calls:
1. One grep read of `abandonStartup`, `removeStrandIfPresent` and `runDirRoot`; my context lacked their bodies, which decide the self-removal race at startup and whether the marker path is absolute.
