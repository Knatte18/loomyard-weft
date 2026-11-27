```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set check of the fixer's change set.

**Left unchecked by a fresh search** (not rerun): F1 and F2 (other places that spell a key name in a message), F3 (other new loom refusals), F6 (seat rows). F8, F13, F16 and F17 I judged from context.

**Findings**

1. **MEDIUM.** F11, `loom-fragment-tooling-names` includes: some briefs that write repository files still lack the rule.
   - `contracts/stencils/loom/loom-fragment-arbiter-fix.md` has the arbiter make and commit a fix "in the task's code, tests and docs", with no rule.
   - `contracts/stencils/gatefix/gatefix-body-card.md` has the card fixer edit code that the fork then commits, and neither it nor `gatefix-frame-cold` carries the rule.

2. **MEDIUM.** F11, the include in `contracts/stencils/burler/burler-step-fix.md` also reaches fixers it should not bind as worded.
   - The same fix step serves the discussion and plan Burler fix rows, whose targets are the decision record and plan files under `_lym`.
   - Those files must name the task, gates, cards and the decision record.
   - The fragment says "A repository file you add or change names nothing of … the board … the task, the Sketch or the decision record … a gate, a tier". It makes no exception for the run's own records, so the plan fixer is told something it cannot carry out.

3. **MEDIUM.** F8, the added rows pass patterns straight into the functions, so the forwarding the finding named is still untested.
   - Untested sites: `internal/beamcli/deps.go:339`, `:342` and `:470`; `internal/beamcli/dobbydeps.go:362` and `:366`; and the two `DescriptionTerms` closures at `internal/loomcli/wiring.go` (the lightweight one, and the full wiring's near `:1055`).
   - Replacing `cfg.ForbiddenTerms()` with `nil` at any of them keeps every new test green. The finding's issue says "nothing fails when the forwarding … is dropped", beam recorders included.
   - The shedrecipe gate row does cover the `Env.DescriptionTerms` → gate forwarding.

4. **LOW.** F13, `PRReview.judge`'s `staleText` and the comment "While it is set, an open pull request holds stale text, so its review does not end".
   - A wake where `pullRequestText()` fails (`textWarned`, description mid-edit or unreadable) leaves `editWarned` false.
   - An approval or `pr-continue` then still ends the review without the texts having been compared.
   - The new wait doc names only the transient edit, so the code matches it; but #228's "never end while they differ" still has this path.

5. **LOW.** F17, new error paths added in `internal/landingshed/describereview.go` have no row.
   - `releaseReviewer`: unreadable ready record, unparsable ready time, failed write.
   - `check`: a `sent` stat error other than not-exist, and a failed `sent` write.
   - `ResetDescribeReview`: a failed remove.
   - F17 was a coverage gap over exactly this entry's error paths.

**Tool calls**
- Listed the stencils that mention commit or edit and lack the new include. My context held only some stencil bodies.
- Searched for the fragment composers, the describe-review timeout wording and the "loads no config" comments. My context did not show the repo-wide hits. Nothing stale turned up in the timeout wording or the comments.
