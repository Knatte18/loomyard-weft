```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the fixer's change set against its surroundings.

1. **BLOCKING**: `gateRefusalReason` in `internal/landingshed/prreview.go`, as used for an ended review (the F6 fix). Its way forward says `lym loom resume` "gates the task head again before the row pushes it", which is false for an ended review.
   - On re-entry, `openPullRequest` runs `pushTask` when `!found || rec.ReviewEnded`, so the refused head is pushed first.
   - Because `head != rec.GatedHead`, it then calls `removePRRecord` and sets `found = false`.
   - `recordPullRequest(ref, head, false)` then writes a new record with `GatedHead: head`.
   - The refused head is therefore pushed, recorded as gated and never gated again. Following the way forward lets it pass PR-Review once approved, which is F6's premise reached through resume.
   - The test row `AGateFailureInAnEndedReviewHaltsAndMailsNothing` only checks the halt, not the re-entry.

2. **LOW**: the new `ResumeEngine` in `internal/testkit/shuttlefake/shuttlefake.go`. The package doc names the kit's opt-in engine (`SignalEngine` "is the kit's opt-in signal-reading engine") but not this second one. It also says nothing about `Engine` having no resume capability, as it does for signal reading.

The other twins I checked agree with the change: the `landingshed` and `loomcli` package docs, `docs/overview.md`, the stencil against the mail hint, the `pr-continue` help and refusal text, `PATTERN-landing`, the beam template and the `MailedHead` record doc.

Tool calls (1 of 6):
- One `git grep`/`git diff` over `pullrequest/record.go`, the beam template, the recipe, `interruptpolicy.go`, the `loomengine` and `configreg` hunks and `docs/overview.md`: the diffs of most of these were not shown to me in changed form, or only in part.
