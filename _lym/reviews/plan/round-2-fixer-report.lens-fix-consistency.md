```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fixer's edits to cards 5, 14 and 16 agree with the sibling text they touch or depend on.

**Finding 1: LOW.** The tripwire row doesn't cover the new start-path ends.
- **Changed text:** card 14's load-turn paragraph now says that with `startPath` set, the relaunch's died and timeout ends go through `classifyDeadlineExpiry`, then `abandonStartup` or a return of `Result{}` and nil. Card 14 step 3 adds "With `startPath` set, a died or timeout end goes the start path's way the load-turn paragraph states instead."
- **Problem:** that is a new set of died, timeout and done return sites. The sibling tripwire row still reads "the tripwire tests pin `stopForRelaunch`'s and the relaunch's died and timeout ends as audited sites, each reached through the completion-signal check". It doesn't name the start-path sites, where `classifyDeadlineExpiry` decides done and `abandonStartup` finalizes. A reader can't tell whether `TestCompletionSignal_NegativeVerdictReturnSites` and `TestCompletionSignal_FileContractCallSites` must list those sites too.
- **Evidence:** card 14's Edit list holds both tripwire tests. `abandonLoad` (`internal/shuttleengine/loadturn.go:294`) is the model the new text copies, and it reaches done only through `classifyDeadlineExpiry`. The row was left as it was.

**Finding 2: LOW.** Card 14's ImpactSummary is stale.
- **Changed text:** the `startPath` parameter added to `relaunchUnderAccount`, and the start path's new return contract.
- **Problem:** the ImpactSummary is unchanged. It still lists only `Wait`'s branch, the load-hold case, `awaitStartup`'s callers, the stop mode and the PATTERN changes. It doesn't mention that the relaunch now ends differently on the start path, or that doc.go paragraphs are rewritten.
- **Evidence:** card 14's `**ImpactSummary:**` line.

Checked and consistent:
- Card 16's calls to `relaunchUnderAccount` now pass `true` on the start path and `false` in `retryAPIError`, which matches card 14's signature. No other card names `relaunchUnderAccount`, as the fixer's grep showed.
- Card 5 drops `battencli#Command` from Edit, and no other card references it.
- Card 5's Extends still lists `internal/shuttleengine/doc.go`. That is untouched by this fix and outside its reach.
- Card 14 and card 16 both rewrite the skill-load hold paragraph and the re-send paragraph. Each card states its own clause and neither contradicts the other.

No further tool calls made.
