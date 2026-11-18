```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism end to end on one concrete case.

**Findings**

1. **BLOCKING — D4/D6: the re-entry bypasses the code that makes a review input.**
   - Case: a PR-Review findings file holds one BLOCKING finding. The verb reroutes to Code-Burler-Review.
   - Evidence: the review-input and new-generation mechanism lives in the Bouncer, not the review row.
     - `internal/shedadapters/amendment.go`: `startNextGeneration` reads `keptReviewInput` and writes the generation record.
     - `doc.go`, lines 290–294: re-entry over a settled verdict re-settles with no spawn, unless `ReviewInput` reports an input newer than the settle, in which case an amendment generation starts.
   - What goes wrong:
     - Rerouting straight to Code-Burler-Review skips the Bouncer. Nothing archives the settled run directory, writes a generation record, or hands the findings to the review row. The step's input is not what the previous step produced.
     - If the edge instead went through the Bouncer, today's path gives an amendment. `AmendmentNote` scopes it to "only the change since the settle". At send-back no commit has landed since the settle, so the review would cover an empty change plus the input. That contradicts D6's "reviews the whole review range".
     - The `ReviewInput` seam reports only the Parent-Align fixer report today.
   - The record needs to say:
     - which row the edge enters;
     - that the PR findings start a full generation rather than an amendment (`cfg.FullGeneration` exists);
     - that the findings become a second review-input source.

2. **MEDIUM — D4: how the row learns of the send-back is unstated, and the verb stopping its own session breaks.**
   - Case: the agent runs `lym loom pr-findings f.md` from inside the PR-Review session.
   - The row waits on the PR record. `pr-continue` signals it by writing a flag into that record (`landingshed/doc.go:27`). D4 says the verb "stops the PR-Review session" and that "the row returns a Stuck verdict with Reroute", but names no record field the row reads on its wake.
   - The row already stops the session on every verdict (`prreview.go:67`). A verb that stops its caller's session kills the agent before the verb can answer.
   - It should be: the verb writes a send-back mark into the PR record, the row's wake turns that into Stuck+Reroute, and the row's existing stop ends the session.

3. **MEDIUM — D4: uncommitted work at send-back is unhandled.**
   - Case: the agent has started a fix and then runs the verb.
   - The row's review end with a dirty worktree is Stuck (`dirtyEndReason`, `prreview.go:920`). That becomes a halt instead of a reroute.
   - D4 should require a clean worktree, or tell the agent to commit or discard first, as `pr-continue`'s steps do.

4. **MEDIUM — D3: one ruling per file or one per check is inconsistent, and Drift-Recheck re-spawns the judge.**
   - Case: a change touches 5 existing non-test files and no test.
   - The judge spawns because there are "changed existing files with no stored ruling". It writes `loosened: []`.
   - The acceptance criterion says a judgement "leaves out a listed file's ruling", which implies one ruling per file. D3's judgement has one entry per loosened check.
   - Nothing is stored for files with no loosened check, so Drift-Recheck finds them unruled again and spawns the judge again. The reuse that D3 and its criterion promise does not hold.
   - The judge would also spawn on almost every run, since nearly every change touches an existing file.
   - The record needs to say what is stored per file, for example one "checked, none loosened" ruling per file keyed by its diff.

5. **MEDIUM — D3: a parent amendment of a loosened check has no shape.**
   - D3 counts a `lym drift amend` cause as `caused`. Today `amend` takes expectation entries keyed by cells (`keys` or `lines`).
   - The record does not say how an amendment names a loosened check, so the Stuck's way forward points at nothing.

6. **LOW — D2: some roles named in D2 may not write their own commit message.**
   - If the gate fixer's or the implementer's commits are made by Go rather than by the agent, the agent cannot add a trailer.
   - The merge fixer commits at Finalize, after both Code-Review and Drift-Recheck, so D3 and D1 never see its trailer.
   - The record should say how a trailer reaches a commit Go writes, or narrow the role list.

7. **LOW — D6: the friction note path on a repeat send-back is unstated.**
   - A second send-back (the reroute cap is 3), or the re-entered PR-Review agent's own note, targets the same per-row note. D6 does not say whether the note is overwritten or appended.

**Per decision**

- D1, case: a widened tolerance with a trailer in a darn run. Purpose met.
- D2, case: a Webster commit carrying a `Loosened-Check:` trailer. Met for agent-written commits; see finding 6.
- D3, case: a widened test limit with no trailer, then Drift-Recheck after a fix. Partly met; see findings 4 and 5.
- D4, case: one BLOCKING finding from the parent's review. Not met as written; see findings 1, 2 and 3.
- D5, case: `run_settings` sets `loom.pr_review_small_max` to 5. Met, since loom keys are already set per task in board `run_settings`.
- D6, case: a send-back writes the note and the re-entered round reads the input. Not met; see finding 1.
- D7, case: Describe writes a tapestry PR body that claims an improvement. Met.

**Calls**

- Grepped `prreview.go`, `prsession.go` and `amendment.go`. My context lacked how PR-Review ends and where the review input is wired.
- Read `amendment.go` lines 1–60 and grepped `shedadapters/doc.go` for settle and amendment. My context lacked the re-entry and generation rules.
- Grepped `gatefixer` for commit, and `landingshed` for the record fields. My context lacked who commits and how the PR record signals the row.
- Grepped the `pr-continue` docs and `gatefixer` files. My context lacked how a verb signals the row.
