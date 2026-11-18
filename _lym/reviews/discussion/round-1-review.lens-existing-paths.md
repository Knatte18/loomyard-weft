```yaml
lens: existing-paths
outcome: reported
```

**Paths traced, per decision:**

- D2: `Loosened-Check:` trailer read by Code-Review over `git log` of the review range — survives (the rubric already reads the range's messages for `Pattern-Departure:`, `contracts/stencils/loom/loom-rubric-code-review.md:79-83`).
- D3: stored-ruling reuse in Drift-Verify (`internal/driftengine/verify.go:245-296`), reapplied to new per-file loosened rulings keyed by diff hash — **breaks** (a trailer commit leaves the diff hash unchanged).
- D3: command diff mode, exit 0 is Done before any judge (`verify.go:69`) — survives as stated ("even with no changed cell", "both diff modes"; the record names the change).
- D3: `lym drift amend`'s `dropCoveredRulings` rewriting the judged record (`internal/driftengine/amend.go:166-174`) — survives (touches only cases/tuples, not new loosened rulings).
- D3: `Verdict` read by `pullRequestText` on every text sync (`internal/landingshed/prreview.go:387-400`) — survives (loosened lines reach the PR body).
- D4: send-back mark in the pull-request record, read by the wait's wake via `RecordChanges` (`prreview.go:561`, `prreview.go:623`) — survives (a record change wakes the wait within `prWaitFloor`).
- D4: `closeRecordedPullRequest` / `openPullRequest`'s `RemoveRecord` on a ReviewEnded record — survives (neither runs between send-back and the wake that clears the mark).
- D4: PR-Review re-entry → `askParentReview` (`prreview.go:343-367`) returns early on `rec.ParentReviewAsked` — **breaks** (parent not re-asked).
- D4: PR-Review re-entry → `ensureSession` (`internal/landingshed/prsession.go:65-103`) resumes `rec.SessionID` — **breaks** vs the criterion "opens a new review session".
- D4: Code-Bouncer re-entry via `reenterUnchanged` / `keptReviewInput` (`internal/shedadapters/bouncer.go:702-735`, `internal/shedadapters/amendment.go:31`) — survives as a stated change (today a newer review input yields an amendment because Code-Bouncer has no `FullGeneration`; D4 names the switch to a full generation for a PR-findings input).
- D4: review input seam set for every code-subject Bouncer (`internal/shedrecipe/entries_bouncer.go:267-270`) wired to the single Parent-Align report path (`internal/loomcli/wiring.go:1164-1166`) — survives (D4 states the seam reports the send-back file too).
- D4: `rerouteCount` resets only on a PR-Review Done or RerouteCapped (`internal/shedengine/reroute.go:16-31`) — survives (the cap counts send-backs across PR-Review re-entries as D4 says).
- D6: `friction.NotePath` non-clobbering stems (`internal/friction/friction.go:120-158`) — survives (a second send-back gets `<id>-2.md`).

**Findings:**

1. **BLOCKING, design** — `decision-record.md` D3 "Drift stores each file's ruling keyed by a hash of the file's diff, and reuses it while that diff is unchanged".
   The ruling depends on the range's `Loosened-Check:` trailers as well as the diff, but the reuse key is the diff alone. D3's own way forward is "a commit carrying the trailer" (possibly with no change) "then `lym loom resume`": that commit leaves the file's diff and hash unchanged, so Drift-Verify on resume reuses the stored `uncaused` ruling and is Stuck again. The trailer way forward never clears the halt, and no acceptance criterion covers a trailer commit clearing an `uncaused` ruling.
   Evidence: `decision-record.md:67`, `decision-record.md:71`; `internal/driftengine/verify.go:246` ("Stored rulings are reused, and the changes left over go to the judge").

2. **MEDIUM, design** — `decision-record.md` D4 "PR-Review again, which finds the open pull request and, with `pr_review_ask_parent` on, asks the parent again".
   `askParentReview` asks once per record and returns early when `ParentReviewAsked` is true; re-entry keeps the record (`recordPullRequest` with `hadRecord` only updates number/URL/base/head/gated head), so the flag stays true and the parent is never asked again. D4 states no reset of the flag on send-back, and no criterion covers the re-ask.
   Evidence: `internal/landingshed/prreview.go:345`, `prreview.go:366`, `prreview.go:445-456`.

3. **MEDIUM, design** — `decision-record.md` D4 and criterion "the PR-Review row re-entered after a send-back opens a new review session".
   The row stops the session on the send-back verdict; on re-entry `ensureSession` finds it not live and starts with `ResumeSessionID = rec.SessionID`, resuming the old conversation that the verb told to end its turn after the send-back — not a new session. The record is silent on clearing `SessionID` or on resume, so the criterion contradicts the existing resume path.
   Evidence: `internal/landingshed/prsession.go:67-82`; `prreview.go:67`, `prreview.go:83`.

Tool calls:
- Read `ensureSession` in `internal/landingshed/prsession.go` — manifest lacked the re-entry session path.
- Grepped `internal/loomcli/wiring.go` around ReviewInput wiring, plus `FullGeneration` and `budget.` gate users — manifest lacked who sets `FullGeneration` and the review input.
- Read the seam selection in `internal/shedrecipe/entries_bouncer.go` — manifest lacked which Bouncer rows get ReviewInput.
