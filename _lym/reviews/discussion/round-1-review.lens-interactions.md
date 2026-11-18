```yaml
lens: interactions
outcome: reported
```

**Pairs checked**

- D2 × D3 (trailer as cause × ruling stored by diff hash): **broken** — a trailer-only commit leaves the file's diff unchanged, so the stored `uncaused` ruling is reused and the row stays Stuck.
- D1 × D2 (rubric accepts trailer × trailer on any range commit): held.
- D1 × D3 (two judges of one loosening, one cause source): held — the trailer is the sole cause in both.
- D2 bound × D4 fix-here (fix-here commits unchecked): held — stated and accepted in D2's bound.
- D3 × D4 (send-back re-entry × Drift-Recheck reuse/re-rule): held for changed files; breaks only through the D2 × D3 defect.
- D4 × D6 (full generation × rubric `## Review input` reading): **broken** — a full generation carries no review input, and the rubric reads one only through the amendment note.
- D4 internal (review input "reports the newest send-back file as well as the Parent-Align report" × one-path seam): **broken**, folded into the D4 × D6 finding.
- D4 × D5 (threshold × setting meaning): held — "more than `pr_review_small_max`" sends back and "at most" fixes here, in both Decisions and criteria.
- D4 × Shed reroute cap: held — `rerouteCount` counts PR-Review's reroute entries since its last Done, and PR-Review never Dones between send-backs.
- D4 × D2 fragment (PR-Review agent commits code): held — it gets the fragment; its fix-here loosenings fall under D2's stated bound.
- D3 × D7 (Verdict lists loosened rulings × Describe figures from the drift verdict): held.
- D3 × D4 Verdict in PR body (`drift_verdict`): held.
- D3 both diff modes × command mode (exit 0 is Done): held — D3 says the judge spawns with no changed cell.
- Constraints × D2–D6 (stencil ownership, fabric boundary, recipe registry, config layers, run records, tolerant records): held.
- Criteria × D3: **gap** — no criterion checks that the stated way forward (trailer commit, then `lym loom resume`) clears the Stuck.
- Criteria × D2: **gap** — Go appending a handoff's trailer lines to a Go-written commit is testable behaviour with no criterion.

**Findings**

1. **BLOCKING · design** — `_lym/discussion/decision-record.md` D3 (lines 67, 71) with D2 (line 42).
   Summary: The per-file ruling is keyed by a hash of the file's diff alone, so D3's trailer way forward never clears an `uncaused` Stuck.
   D2 says a forgotten trailer is added in a later commit "with no change if need be", and D3's way forward is "a commit carrying the trailer … then `lym loom resume`"; that commit leaves the file's diff unchanged, so Drift-Verify/Drift-Recheck reuses the stored `uncaused` ruling, spawns no judge, and halts again. The parent can only escape by reverting the loosening, which defeats D2.
   Fix: key the ruling on the diff together with the range's `Loosened-Check:` trailers (or never reuse an `uncaused` ruling), and add a criterion that a trailer-only commit plus resume clears the Stuck.
   Evidence: `decision-record.md:42`, `:67`, `:71`, `:158`, `:161`.

2. **MEDIUM · design** — `_lym/discussion/decision-record.md` D4 (lines 101–102, 111) with D6 (line 125).
   Summary: D4 says a PR-findings input starts a full generation carrying a review input and its rationale says that mechanism exists, but in the code a full generation drops the review input.
   `startNextGeneration` reads and keeps the input only when it starts an amendment; a full generation "carries the number alone". The rubric reads a review input only "When the amendment note names a review input", and a full generation renders that note as `none`. So D6's rubric reading has no channel. The `ReviewInput` seam also returns one path, while D4 asks it to report both the send-back file and the Parent-Align fixer report, and the amendment-vs-full choice then depends on which input is newest — unstated.
   Fix: state the new mechanism (a full generation that carries a review input, the channel naming it to the reviewer, and how two inputs coexist or which wins) and correct the rationale's "only the mark, the verb and the second input source are new".
   Evidence: `internal/shedadapters/amendment.go:19–50`, `internal/shedadapters/bouncer.go:215–225, 729`, `contracts/stencils/loom/loom-rubric-code-review.md:133–141`.

3. **LOW · consistency** — `_lym/discussion/decision-record.md` Acceptance criteria (lines 156–174) vs D2 (line 44).
   Summary: D2 decides "Where Go writes a role's commit, the role's handoff carries the trailer lines and Go appends them", testable behaviour with no criterion; D2's only criterion is the `Review:` fragment check.
   Fix: add a criterion that a Go-written commit carries the `Loosened-Check:` lines its role's handoff gives.
   Evidence: `decision-record.md:44`, `:157`.

Tool calls:
- Read `internal/shedadapters/amendment.go` lines 15–60 and grepped `FullGeneration`/`ReviewInput` in `internal/shedadapters/bouncer.go` — my context had only comment summaries, not how a full generation treats the review input.
