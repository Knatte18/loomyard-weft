```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. No list was asked for.

1. LOW — Card 12, the line added to `**Edit:**` (`docs/overview.md`) and the new Intent sentence (`docs/overview.md`'s batten paragraph … names the adopt check's green end in its place, beside `lym beam land`).
   Summary: card 12's `**ImpactSummary:**` now leaves out the overview edit.
   Evidence: `12-check-green-end.md:76` still reads "landingshed's owner refusal and battenshed's doc name the green end as what resumes and starts". The card now also edits `docs/overview.md` for the same sentence, but the summary does not mention it.

2. LOW — Card 13, the new Intent sentence at line 50 ("The failed notice `runCheck` posts for a failed tier loses its `fix it, then run <re-run> in the prime` way forward…").
   Summary: card 13's `**ImpactSummary:**` still lists only the no-verdict notices as reworded.
   Evidence: the summary reads "runCheck gains one seam at a failed end and its no-verdict notices name the re-run". The fix also rewords the failed-tier notice, and the summary does not mention it.

Checked and found consistent:
- Card 6 lines 37, 55, 65, 84 and 89 use `battencli.Cancel` and its end-only error in a way that still matches the new composition in card 4.
- Card 5's notice prefix and `tierNotice` changes do not conflict with card 13's reworded failed notice.
- The edits to `docs/overview.md` from cards 5, 11 and 12 touch separate paragraphs.
- Card 11's new rule (`ahead` warns only with `Moved` unset) matches its new and reworded `TestAdopt` rows. `00-overview.md` and `14` do not refer to `reconcileWarnings`.

Tool calls:
- One grep over `_lym/plan/*.md`, to find sibling cards that refer to `Cancel` and its end-only error, the failed notice, `docs/overview.md` and `reconcileWarnings`. My context held only cards 4, 11, 12 and 13.
