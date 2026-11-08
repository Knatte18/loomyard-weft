# Burler fixer: act on a parent ruling that arrives mid-round

A burler fixer is told that `decision`-class findings go to its parent and that it neither fixes nor reports them.
When the parent's ruling on one arrived mid-round, as parent mail or a decision notice, that instruction conflicted with the ruling.
The fixer-report had no section for the resulting work, and a fixer that found the ruling mistaken or redundant had no way to tell the parent.
This change covers all three gaps in the fixer's instructions.

Closes #84

## What changes

- `burler-step-fix.md` gains a parent ruling rule.
  A ruling overrides the "neither fix nor report" rule: the ruled finding becomes the fixer's work, with the ruling in place of its `fix` key.
  A ruling that arrives after the fixer-report is written adds its entry to the report.
- The fixer carries the ruling out inside its normal write surface and git discipline.
  A part that would need to leave the write surface is left undone, with the reason.
- A mistaken or redundant ruling is flagged instead of carried out.
  The fixer posts once with `lym creel post --parent "<text>"` and no `--expect-reply`, so the round goes on.
  With no parent recorded, the fixer-report entry is the only flag.
- The fixer-report gets a `Rulings` section, with one entry per ruling.
  Rulings are never recorded under `Disputed` or as deferred.
- `burler-template-fixer.md` keeps "send no message about the review" with that one exception.
- `routedDecisionsLine` in `internal/burlerengine/prompt.go` and the package doc in `internal/burlerengine/doc.go` now say the routing holds until the parent rules.

## Design decisions

- A ruling counts as redundant when the target already holds what it asks for.
  It counts as mistaken when the code shows its premise false, or the change breaks a test, caller or `PATTERN` entry.
  A ruling the fixer merely disagrees with is carried out.
- The flag is a one-way creel post rather than a deferral.
  The issue allowed either channel; a post reaches the parent during the round, while a deferral surfaces only after it.

## To check by hand

- The change is prompt text only, and no tests were added or run for it.
  Read the new rule in `burler-step-fix.md` as a fixer would, and check that `routedDecisionsLine` reads coherently with it.
