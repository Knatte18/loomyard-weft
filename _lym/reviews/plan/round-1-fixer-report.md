# Round 1 fixer-report — plan

Every finding's premise held against the code at HEAD, and all six are fixed.
`lym webster validate` reports only `plan-unapproved` as blocking, which is the baseline for an unapproved plan, and informational `create-new-unit` findings.

## Changes

- **F1** (BLOCKING), fixed as proposed, in `_lym/plan/04-machine-name-set.md`.
  - The Create lines for `TestNextFile` and `TestHubOfDir` are deleted, and so are their Tests bullets.
  - `TestName_Set` gains `NoFileTakesTheHostnameNameAsOld`, `AlreadySetWritesNothing`, `NoHostnameAddsNone`, `NewNameMovesToTheEnd`, `CwdContainerDirectoryIsSwept` and `PlainCwdAddsNoHub`.
    The two cwd rows set `cwdHub` to `hubOfDir` itself.
  - The bullet now states that each row that writes the file asserts the written `names`.
    `OwnFormerNamePassesTheCollisionCheck` (host `box`, `["neve","box","osl46"]`) and `EmptyEntryDropIsReported` (`{"name":"box","names":["","box"]}` gives `["box","osl46"]`, one dropped) gain concrete expected names, which carry the cut `TestNextFile` rows.
  - The worktree-subdirectory note stays.
- **F2** (BLOCKING), fixed as proposed, in `_lym/plan/01-owner-replace-set.md`.
  The `ReplaceSetLosesToACompetingOriginWrite` bullet is deleted.
  The Intent keeps the re-read sentence and adds that `CompetingWriteLosesToOrigins` pins the Lost report on the shared re-read, while `ReplaceSetSelectsAfterThePull` pins that replace-selected slugs enter it.
- **F3** (MEDIUM), fixed differently, in card 1's Intent.
  The three docs at `ownerset.go:43`, `ownerset.go:72-73` and `doc.go:183` gain the replace set's exception.
  - The proposed `Takeover` wording "without it, or a replace set naming that recorded owner, such an entry is left alone" is not used, because it reads as if the replace set also leaves the entry alone.
  - The fix-proposal self-check found that a bare "unless Replace names its recorded owner" implies a replace write reports every entry owned elsewhere in `Lost`.
    So the `Takeover` and `SetOwners` sentences now say the Lost rule covers entries the request names, and that `Replace` selects only entries whose recorded owner it names (`writeOwners` adds to `lost` only at `ownerset.go:179` and `:195`).
- **F4** (LOW), fixed differently, in card 4's step 4, the `hubOutcome` field list and a new `TestName_Set` row `UnpushedReadErrorIsNotedAndSwept`.
  - The proposal put the `unpushed` error in the checked-locally detail.
    The same step says that detail marks a hub `checked locally only`, so a hub whose pull succeeded would be reported as checked locally.
  - The error goes instead in its own unpushed-read note on `hubOutcome`, and it reads as not ahead.
    The row asserts that a hub whose pull succeeded is swept, carries the error in that note and is not marked checked locally.
- **F5** (LOW), fixed as proposed, in card 4's step 7 and the `FailedHubIsReportedAndOthersSweep` row.
  The outcome carries the error's text, and its way forward field is the re-run.
- **F6** (NIT), fixed as proposed.
  The `fakeHubs` text names the per-hub seams `pull`, `unpushed`, `readOwners` and `setOwners`, and it also gained the unpushed error among the values it returns.

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-affected-set: no findings; no finding left unchecked.
- fix-proposal, MEDIUM, F4's error in the checked-locally detail falsely marks a pulled hub as checked locally: fixed with own change (its own unpushed-read note; see F4 above).
- fix-proposal, LOW, F3's exception implies a replace write reports every entry owned elsewhere in `Lost`: fixed with own change (scoped wording; see F3 above).

## Disputed

None.

## Deferred

None.

## Rulings

None.
