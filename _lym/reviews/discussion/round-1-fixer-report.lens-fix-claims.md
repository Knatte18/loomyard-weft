```yaml
lens: fix-claims
outcome: reported
```

I checked every claim the fix added or rewrote against the code at HEAD. Two are wrong; the rest hold.

**MEDIUM — D6's "landed" test counts a Finalize that stuck as landed**
- Changed text: D6, "Only a tag with no index row whose history holds Finalize gets a line derived from the tag's tree and the landings join…, marked as derived". The reviewer's F6 fix proposed this rule.
- Summary: a Finalize history entry is also recorded when Finalize ends Stuck, so a run that stuck at Finalize before its merge and was then cancelled gets a derived landing line, which F6 was meant to stop.
- Evidence:
  - `internal/shedengine/reroute.go:49` and `:70` add a `HistoryEntry{Producer: def.Name, Outcome: Stuck, …}` to the history.
  - `internal/reportengine/runs.go` `readLoomRecords` only asks whether some entry has `entry.Producer == loomshed.NameFinalize`, whatever its outcome.
  - So "history holds Finalize" does not mean the run landed. The new criterion "a cancelled run's tag is listed as not landed" fails for a run cancelled after a Stuck Finalize.

**MEDIUM — D4's rationale says Parent-Align runs a fix review, and it does not**
- Changed text: D4 rationale, "The fix review that Finalize and Parent-Align run over an over-cap fixer change writes its findings only to scratch". The wording comes from the F9 summary.
- Summary: only Finalize runs a fix review.
- Evidence:
  - The only non-test caller of `runFixReview` is `internal/landingshed/finalize.go:1130`, with row `finalizeName`.
  - In `internal/landingshed/parentalign.go`, the over-cap path (`:310`, `:411`) sets `fixerHaltOverCap` and calls no fix review.
  - `parentalign.go:70` itself says review-capped marks "an over-cap change at Finalize".

**LOW — D4's rationale gives the wrong causes for the review round count repeating**
- Changed text: D4 rationale, "The attempt record's spent count and review rounds repeat within a row, since a new row entry, the operator's removal of the record, an escalate and a failed start each reset or take one back".
- Summary: an escalate and a failed start touch only `Spent`, not `ReviewRounds`. The reset that is missing from the list is an operator acceptance: a change past the accepted head zeroes `ReviewRounds`.
- Evidence:
  - `fixerrun.go:296` does `s.rec.Spent--`, and `fixerrun.go:150` restores the original record.
  - `finalize.go:1124-1126` does `rec.AcceptedHead, rec.ReviewRounds, rec.Findings = "", 0, nil`.
  - The conclusion that numbers repeat still holds.

Claims I checked that hold:
- D1: `internal/state` writes through `fsx.AtomicWriteBytes` (`internal/state/state.go:59`, `:99`), so the `.tmp-*` rule covers the D3 copies.
- Constraints: `pr-findings` commits through `pullRequestRecordCommitter` (`prfindings.go:344`), and `pattern/PATTERN-run-fsm.md:27` says the pull-request record keeps its own synchronous push.
- D2: the sha256 hash matches `prfindings.go:222`.
- D5's ordering claims (after the parent push and any archive, before `pushRecords`, before `dropBeamRecord`) match `finalize.go`.
- D7 matches `cleanupremotewarp.go:336-379`.
- The `PATTERN-pair-remote` text matches `PATTERN.md:24`.
- The plan-writer notes name real symbols.

Tool calls I made:
1. A grep for `runFixReview` and `fixReviewTable` callers, the Parent-Align files, `internal/state`'s use of `fsx`, `PATTERN-run-fsm`'s note on the pull-request push, and the `pr-findings` commit wiring. My context had `runFixReview`'s body but neither its callers nor those files.
2. A read of `parentalign.go`'s fix-review use, `finalize.go:1120-1135`, and where shedengine adds history entries. My context did not show how a Stuck producer enters the history that `readLoomRecords` checks.
