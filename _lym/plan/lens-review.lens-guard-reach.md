```yaml
lens: guard-reach
outcome: reported
```

guard-reach lens report on the result-checks plan (cards 1–12): six findings, one BLOCKING.

**1. BLOCKING — Card 7 (Bouncer review inputs, D4): a send-back after an amendment approved with no review never reaches the reviewers**
- **Path:** card 7 only handles a full input in `Bouncer.startNextGeneration`. `Call` reaches that only through `if n > 0` → `settledGeneration(n)` → `settledEntry` → `entryNewGeneration` (`internal/shedadapters/bouncer.go:449-458`).
- **How the state arises:** set `review_amend_max_bounces: 0`, then let a Parent-Align amendment run. `startNextGeneration` archives the run directory and settles through `approveWithoutReview` (`internal/shedadapters/amendment.go:42-57`). That leaves a live settle record at round 0 and no review files (`internal/shedadapters/settlerecord.go:189-214`).
- **What then breaks:** the PR-Review send-back re-enters Code-Bouncer. `reenterUnchanged` returns handled=false because the full input was kept. `ResolveRound` then gives `n == 0`, so `Call` goes to the seed path (`bouncer.go:480+`) and never reaches `startNextGeneration`.
- **Effect:** no generation record holds the send-back path, `ReviewInputNote` renders `none`, and the reviewers are never told of the findings file. Criterion D4 ("its review input names the send-back file") fails.
- **Fix:** start the full generation from `reenterUnchanged`, or from the `n == 0` entry, whenever a full input was kept. Add a `TestBouncer_Amendment` row in which an amendment with allowance 0 comes before the send-back.

**2. MEDIUM — Card 3 (D3 listing): the loosened-check listing is never reached when the drift scope is off, deferred or waived**
- `DriftVerify.pass` ends the pass Done or Stuck before `checkAndJudge`/`passCommand` in these cases (`internal/driftengine/verify.go:125-145`):
  - no scope record
  - `ScopeOff`
  - `Deferred`
  - `WaivedAt` set
  - a capture that never counted
- With `drift.enabled` on, a widened limit in an existing test is therefore never listed or judged in those runs.
- Card 3 names only `drift.enabled` as the bound, and D3's Bound names only that too.
- **Fix:** run `listFiles` before those returns, or state and bound the skip.

**3. MEDIUM — Card 9 (`settleSendBack`, D4): a mark with no own history entry has no test row**
- On a run's first send-back, PR-Review has no history entry of its own until shed persists one.
- A crash between the reroute verdict and shed's persist therefore resumes with a mark and no own entry.
- Card 9's rows cover an own Stuck entry, a reroute of another file and a `RerouteCapped` entry, but not an absent entry. That is the nil branch of "newest own entry", which no row reaches (glyph `plan:internal/landingshed#PRReview.settleSendBack`).

**4. MEDIUM — Card 8 (`prFindingsVerb` step 4): the "mark already held" branch has no test row**
- Step 4 answers the send-back again and writes nothing when the record already holds a mark.
- This is reachable when the agent runs the verb twice before its turn ends.
- None of `TestPRFindingsVerb`'s listed rows reaches it.

**5. MEDIUM — Card 8 (`prFindingsVerb` step 3): the no-record refusal has no test row**
- Step 3 refuses a run with no pull-request record.
- `PATTERN-refusal-way-forward` requires a test that reaches each new refusal and asserts its message, and no `TestPRFindingsVerb` row does.

**6. LOW — Card 3 (verdict): some halts leave the verdict without the run's loosened checks**
- `checkAndJudge` halts on unexpected cells (`verify.go:258-267`) and on stored no-match rulings (`verify.go:283-288`), and records the check, before any listing.
- `CheckResult.Loosened` stays empty for those checks, so the verdict's `- Loosened checks:` lines leave out loosened checks the run does have. Card 3 says the halt comes "after" those but does not say the listing runs first.

Tool calls beyond my context (4):
- grep for the Bouncer entry dispatch in `bouncer.go`: to trace how a re-entered Code-Bouncer reaches `startNextGeneration`.
- `bouncer.go:387-470` and `780-835`: the `Call` entry order and `settledEntry`, which my context lacked.
- `liveSettleRecord` and the Bouncer skip seam: to learn when a settle record counts as live and whether zero rounds skips before re-entry.
- `bouncer.go:470-500` and `entries_bouncer.go:164-183`: to confirm the `n == 0` seed path and the zero-rounds skip.
