# Round 3 fixer-report — discussion

Every finding of `round-3-review.md` is fixed; every premise held against the code.

## Changes

- **F1** (fixed differently, after the self-check): D2's fresh-launch bullet replaces the closed list with a rule.
  The rule names every record of the worktree's lym state that stores a provider session id its owner later resumes, each read through its owner module's exported reader, with the orch state record as an example.
  The evidence names only the sampler row's record and the PR-Review session record, not the gate fixer, webster or treadle the fix key proposed.
  Only `internal/loomshed/samplersession.go`, `internal/landingshed/prsession.go` and `internal/orchcli/start.go` set `ResumeSessionID`, and no file under `internal/gatefixer`, `internal/websterengine` or `internal/treadleengine` resumes a session.
  The rule says "wherever its owner keeps it for the worktree", not "under the worktree's anchor", because the sampler and PR-Review record paths are told to their modules (`RecordPath` deps), and where they live is not settled here.
  The bullet now opens "With `claude.accounts` listing an account", to agree with the empty-list bullet, which searches nothing.
  The "reads reed's or orch's records" sentence now reads "any owner module's records".
  Added the PR-Review/sampler acceptance bullet under D2.
- **F2** (fixed differently, after the self-check): the D2 resume-check bullet and D6's first bullet now say the live-holder test reads the registries of every known directory as one set, by D6's combining rule, and refuses on any live entry.
  The fix key proposed the resolved directory's registry alone.
  That misses a live holder when one session id has transcripts in two directories, a case D6's `LocateSessions` bullet allows.
  The bound now states that an unreadable registry warns and the check proceeds, as `resume.go` does today, in place of "never on any account".
  Added the D6 acceptance bullet, which says the refusal holds also when the check resolved another directory.
- **F3**: D6's id-only bullet states the combining rule: one set of entries, no doubt from a missing registry directory, doubt from an unreadable one, then the single-registry rule.
  Added the D6 acceptance bullet.
- **F4**: a gone recorded directory refuses a launch, fresh or resume, whatever `claude.accounts` holds.
  The empty-list bullet, the resume-refusal bullet's way forward and the resume-check bullet's empty-list sentence now say so.
  The gone-directory acceptance bullet gains the empty-list row.
  The support log's rejected alternatives record why exporting nothing was not chosen.
- **F5**: D3's counter paragraph sets the count, whether that raises or lowers it, defines the no-reference case as 0, says a kept count is overwritten, and drops the sentence about kept counts.
  Step 5 says "set counts".
  The D3 acceptance bullet is reworded to match.
- **F6**: the D3 criterion is restated by count over weight, and the sentence "never takes two picks in a row" is removed.
- Support log: a round-3 line under Review rounds.

## Self-check

- fix-consistency, MEDIUM: the D3 acceptance bullet claimed "lowered" for a lowered weight. Fixed as proposed ("set to that value the same way, whether that raises or lowers it").
- fix-consistency, LOW: the untouched sentence still named only reed's and orch's records. Fixed as proposed.
- fix-consistency, out-of-lens note: "under the worktree's anchor" was unverified. Fixed with my own change ("wherever its owner keeps it for the worktree"), since the record paths are told deps.
- fix-claims, MEDIUM: the gate fixer, webster and treadle do not resume. Fixed: dropped from the evidence; the only `ResumeSessionID` setters are the sampler, PR-Review and orch.
- fix-claims, MEDIUM: reed's strand list is not live-only (`internal/reedengine/lifecycle.go` `planResumeLaunches`). Fixed: now "reed drops a strand once its work is done".
- fix-claims and fix-proposal, MEDIUM (merged): "never on any account" overclaimed. Fixed as fix-proposal proposed: the live-holder test spans every known directory's registry, and the bound states the unreadable-registry warning.
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None reached me.
