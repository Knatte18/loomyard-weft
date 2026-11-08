# Burler fixer: act on a mid-round parent ruling, report it, flag bad ones

When a burler fix row routes a review's `decision`-class findings to the run's parent, the fixer's work line tells it that it neither fixes nor reports them.
The parent's ruling on such a finding can still reach the fixer during the round, as parent mail or a LOOM decision notice that the parent directive says to act on.
Until now nothing settled which of the two instructions wins.
The fixer-report had no section for work done on a ruling.
The fixer seat also banned every message about the review, so a fixer that found a ruling redundant or mistaken could raise it only as a deferral.

## What changes

- `contracts/stencils/burler/burler-step-fix.md` gains a **Parent ruling rule**.
  A ruling on a `decision`-class finding of this review overrides the neither-fix-nor-report rule:
  the ruled finding becomes the fixer's work, with the ruling in place of its `fix` key, until the fixer ends its turn.
  A ruling that arrives after the fixer wrote its report adds an entry to that report.
  The fixer carries the ruling out inside its write surface.
  If the target already holds what the ruling asks for, or the code shows the ruling's premise is false or its change breaks a test, caller or `PATTERN` entry, the fixer leaves that part undone.
  It then flags the ruling to its parent once with `lym creel post --parent` and no `--expect-reply`, naming the finding id, the ruling and the evidence.
  A fixer that only disagrees with a ruling still carries it out.
  With no parent recorded, the report entry is the only flag.
- The fixer-report rule adds a `Rulings` section, empty when no ruling arrived.
  Each entry names the finding id, what the ruling asks, and what the fixer did: the changes it made, or the part it left undone with the reason, the evidence and whether it told the parent.
  Rulings never go under `Disputed` or deferred.
- `contracts/stencils/burler/burler-template-fixer.md` relaxes its no-message rule.
  The fixer may now send the one flag that the parent ruling rule names.
- `internal/burlerengine/prompt.go`: `routedDecisionsLine`, the routed work line, now ends with an exception for a ruling that arrives this round and points to the new rule.
- `internal/burlerengine/doc.go` documents the ruling path next to `RunOpts.DecisionsRouted`.

## Design decisions

- **The flag is a creel post with no reply expected.**
  The parent directive's `--expect-reply` form ends the asker's turn, which for a fixer would end the round.
  A plain post lets the round finish while the parent reads the flag.
  The `Rulings` entry records the same evidence for later rounds and for a run with no parent.
- **Only rulings on this review's `decision`-class findings fall under the rule.**
  Any other parent message keeps following the parent directive as before.
- **The engine-written skipped reports keep their sections unchanged.**
  A skipped fix half has no session, so no ruling can reach it.
