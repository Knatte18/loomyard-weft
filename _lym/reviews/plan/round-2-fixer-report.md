# Round 2 fixer-report — plan

## Changes

- **F1** (MEDIUM, design), fixed as proposed.
  Card 15's Intent now says the refusal of `lym tender resume <slug>` for a slug only a `bolt-hold` entry covers is judged after the verdict and before `req.Clear`, so a refused resume clears no pause intent and pushes nothing.
  `TestResume` gains the row: a refused resume of a `bolt-hold`-only slug whose run carries a pause intent leaves `Clear` uncalled and the intent standing.
- **F3** (MEDIUM, design), fixed as proposed, plus the twin in card 20.
  Card 21's trigger list states that `PairPresent` is a path check while the hand-over removes only a registered linked worktree.
  A run handed over by a successful catch-up through the `hand_over` trigger owes no second catch-up from it while it still lists as `hand_over`.
  The daemon keeps such slugs in `handedOver`, which the catch-up's success does not drop and which forgets a run once it stops answering `hand_over` or is no longer listed.
  The daemon field list names `handedOver`, and `TestRun_CatchUp` gains the row: a listed `hand_over` run whose pair stays present after a successful catch-up owes the catch-up once.
  Card 20's `DobbyPairSlugs` now names its predicate, the same path check as `RunView.PairPresent`, so the review's cross-card claim also holds in card 20 itself.

## Self-check

Every fork returned a report; the lens reports sit beside this file.
The two `isBeam` findings (fix-consistency MEDIUM, fix-proposal LOW) were merged.

- fix-claims, MEDIUM: card 23 named a nonexistent `orchcli.startReport`. Fixed as proposed: it now names `orchcli.BeamUpReport.OwnerPauses` (`internal/orchcli/start.go:67`, `:89`).
- fix-consistency, MEDIUM, merged with fix-proposal, LOW: a failed or nil `isBeam` read was unspecified. Fixed with my own change, since neither lens proposed one: a failed read ends nothing for the entry and gives the "could not be ended" warning with the read's error; a nil `isBeam` reads no entry as a beam. `TestStart_ReconcilesTheHubFirst` gains the failed-read case.
- fix-consistency, LOW: `internal/boardengine#ReadOwners` was missing from card 23's `**Uses:**`. Fixed as proposed.
- fix-consistency, LOW: card 23's ImpactSummary was stale. Fixed as proposed: it now says `orchCLI` swaps `pauseOwned` for `endOwned` and gains `isBeam`.
- fix-affected-set, LOW: `handedOver` forced an edit to `Run`, which builds the daemon's per-run maps (`internal/dobbyengine/loop.go:208`). Fixed: `internal/dobbyengine#Run` is under card 21's `**Edit:**`, and the Intent says `Run` creates the map.
- fix-affected-set, LOW: the comment on `startHarness`'s fields (`internal/orchcli/start_test.go:162`) still described the owner pause. Fixed: `internal/orchcli#startHarness` is under card 23's `**Edit:**`, and the test text says the comment describes the end.
- fix-proposal, MEDIUM: under F3's shared predicate, an unregistered leftover directory entered the catch-up's scope, and `FetchPairTips` would fail in it and stall the dobby. Fixed with the lens's second option, in card 20: a pair directory that is not a registered linked worktree (as `Teardown.TaskWorktreeGone` reads it) counts as no pair here for `FetchRun`, `Ahead` and `PushOwed`. `TestHubCatchUp` gains that row. I did not take the lens's first option, scoping by registration, because the trigger's set and the catch-up's scope would then differ again, which is F3's own premise.

The self-check added two `**Edit:**` targets and one `**Uses:**` entry. `lym loom validate-plan` over the plan on disk answered `ok: true`, with only informational `create-new-unit` findings on card 1.

## Disputed

None.

## Deferred

None.

## Rulings

- **F2** (MEDIUM, decision), ruled by the parent through the decision-record entry "Card 23's bring-up ends owned-elsewhere runs through an endOwned seam". The ruling asks for:
  - `endOwned func(location *lymcwd.Location, slug string, endDobby bool) error` filled by `pairteardown.EndSubjectProcesses(prime, slug, endDobby)`, with a nil seam skipped and warned;
  - `isBeam func(slug string) (bool, error)` filled from `boardengine.ReadOwners(...).Entries[slug].Beam`;
  - `owner_pauses` and both `OwnerPauses` fields keeping their names and shape;
  - the two warnings in ended wording;
  - the two tests asserting the seam's calls with the flag and the reworded warnings, with `startHarness`'s fields keeping their names.

  All of it is in card 23's bring-up paragraph and test text.
  One part was flagged instead of carried out literally. The ruling put the fill "where pauseOwned is filled today (installedOwnerPause and its install path)", but card 23 deletes that path, and `orchcli` already imports `pairteardown` (`internal/orchcli/cli.go:23`) and `boardengine` (`start.go:19`). Both seams are therefore filled directly in `wireCLI`, where `pauseOwned` is assigned today (`cli.go:278`).
  I told the parent (creel #1533), and the parent agreed (#1534).
