# Fixer report — discussion round 2

## Changes

All changes are in `_lym/discussion/decision-record.md`.

- F1 (MEDIUM), fixed as proposed, without the proposal's "(and so from `lym s`)":
  D11 gains a second entry point, the foreign-owner refusal of `lym beam start`.
  When the subject is not under adopt (D4), it runs the same fabric archive call under the slug's teardown lock before it refuses, and a failed archive is named in the refusal so a re-run retries it.
  The D11 title no longer names the dobby alone.
  The Bound line now says a machine whose dobby is not running archives nothing until `lym beam start` runs there.
  The parenthetical was left out because D10 runs `lym adopt --here` before `beam start`, so `lym s` on the old owner takes the subject back and never reaches the refusal.
  The Bound states that path instead: that adopt's local-commit stop meets the tips (`internal/adoptcli/adopt.go:747`, `:791-796`).
  A new D11 acceptance criterion covers the beam-start archive.
- F2 (LOW), fixed as proposed:
  D5's `pause` step turns each adopted beam's own-scope `OwnerCaller` entry into an `AdoptCaller` entry in place, writes none for a beam without one, and lists the converted slug among the slugs it paused.
  D3 notes that this entry, which the dropped settled step used to lift, becomes an adopt pause.
  A new D5 criterion covers the case.
  It names the run report and `lym tender resume <beam>`, but not `lym beam status`, because D9's adopt section reads the record and shows no pause entries.
- F3 (LOW), fixed as proposed: D2's field list now stores each done step's outcome, the part of the end envelope that step contributes.
- F4 (LOW), fixed as proposed, with the exceptions reordered after the self-check (below):
  the D5 criterion excludes front tasks, and it gives no entry to a slug holding an entry of any caller but `owner`.
- F5 (LOW), fixed as proposed: the D2 discard criterion now covers both interleavings.

## Self-check

- fix-consistency, LOW: the D11 sentence "The archive is one fabric call the dobby's deps wire" named only the dobby. Fixed as proposed: it now names the dobby's deps and `lym beam start`.
- fix-proposal, LOW: the F4 criterion listed the other-caller exception before the `owner` conversion, and `owner` is itself another caller. Fixed: the criterion now converts the `owner` entry first, and only an entry of any caller but `owner` keeps its slug out.
- fix-proposal, LOW: it was unclear what the converted beam-slug entry holds. Fixed with my own change, since the lens proposed none: D5 says the entry covers the beam pair slug alone, as the `owner` entry did, and the beam's runs and tasks carry their own entries.
- fix-proposal, note (no severity): the lens doubted the D11 Bound's local-commit-stop claim but did not verify it. It is merged with the fix-claims MEDIUM below.
- fix-claims, MEDIUM: "that adopt's reconcile meets the tips through its local-commit stop" was called false for a tip that is only ahead of origin. Disputed, premise false: adopt's own `stoppedBranches` lists `fabricengine.BranchAhead` as a stop (`internal/adoptcli/adopt.go:747`), and `takeOverSlug` stops the entry on it, naming `--keep-local` and `--drop-local` (`:791-796`). The lens read `fabricengine`'s `Stops()`, which is not the list adopt uses. fix-way-forward checked the same code and confirms the claim.
- fix-claims, LOW: "It pauses no run" was wrong, because `lym beam start`'s prepare owner-pauses owned-elsewhere entries before it refuses (`internal/orchcli/start.go:457`). Fixed: the sentence now says prepare has already owner-paused those runs, as today.
- fix-claims, LOW: the run report names the converted beam slug only if the record lists it as paused. Fixed: D5 now says the record lists a converted beam slug among the slugs it paused.
- fix-way-forward: no findings.

## Disputed

- fix-claims MEDIUM (self-check), premise false: evidence is under Self-check above (`internal/adoptcli/adopt.go:747`, `:791-796`).

No review finding is disputed.

## Deferred

None.

## Rulings

None.
