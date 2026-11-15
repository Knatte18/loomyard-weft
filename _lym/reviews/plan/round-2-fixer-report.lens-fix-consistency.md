```yaml
lens: fix-consistency
outcome: reported
```

Checked how the fix's changes fit with sibling cards, criteria and cross-references.

- **MEDIUM**: card 3's `Verdict.Refusal` paragraph, the new `Landed` wording. The new text describes "removing its pair with `lym fabric remove <slug>`… releasing the entry with `lym board set-status …` … as the done-slug refusal of `lym batten run` words it". That disagrees with the refusal it cites and with card 10. `doneSlugRefusal` (`internal/battencli/arm.go:848-853`) names `lym batten cleanup --apply`, the run directory and the leftover branch. It names neither `lym fabric remove` nor a board release. Card 10:30 says the same refusal "names the cleanup and the leftover branch to delete". The two `Landed` ways forward now name different steps, and the "as … words it" cross-reference is false. Fix: drop the "as the done-slug refusal words it" clause, or align the steps with `doneSlugRefusal`/card 10.
- **LOW**: card 11, the `doc.go` bullet: "`doc.go`'s pass-planning paragraph says the act rule decides". After the fix, `planPass` also gates on `run.CancelMarker` beside the act rule, and the `planPass` doc bullet names the cancel marker as a cancel under way. The `doc.go` paragraph still credits the act rule alone, so it reads stale against the planner's gate. Fix: have that paragraph also name the cancel marker (a cancel under way, the lock included) as the planner's own gate.
- **LOW**: card 11, `CancelMarker` wording across the bullets. The fixed bullet says `CancelMarker` sees the held cancel lock before the intent is written (card 8). The untouched bullets still tell the `strandedEnded`/`readWaits`/`runNotices` comments and `doc.go`'s other paragraphs to name the cancel marker as "the unfinished cancel intent" alone. Card 8:74 and card 11:31 say it reads either an intent or a held lock. The twin wording was left narrower than the reason the fix now rests on. Fix: word those as "a cancel under way: an unfinished cancel intent or a cancel lock held elsewhere".

Tool calls:
- grep of `_lym/plan/*.md` for `board list`, `CancelMarker`, `PastParentMerge`, `followup.go`, `childRestepped`: to find sibling cards outside cards 3, 5 and 11, which my context lacked.
- grep of cards 9, 10 and 14 for the done-slug and `Landed` wording, and of card 11's cancel-marker lines: card 10's refusal text was not in my context.
