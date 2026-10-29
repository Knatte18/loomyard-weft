# Fixer report — discussion, round 1

F4 is `decision`-class and went to the parent; this report leaves it out.

## Changes

- F1 (BLOCKING): D10 now names the mechanism.
  It lists why none of `readAgentActivity`'s four live waits fits.
  It then has `readAgentActivity` also count a child status running at `PR-Review`, beside `childAtBeamAwait`, for as long as the status reads that.
  The words "of a kind battenshed's quiet check already reads" are gone, and the rationale was rewritten to match.
  The D10 criterion names the read.
  The support log's Turn 4 and Q5 were updated.
- F2 (BLOCKING): D9's merged Done now requires the merged head to equal the record's gated head.
  A new merged-with-an-ungated-head halt names both heads and gives its way forward.
  D8's bound now states that D9's merged ending and D11 land only the gated head, and that any other merged head halts.
  A D9 criterion covers an ungated head merged on GitHub.
- F3 (BLOCKING): fixed differently from either option on its own.
  The launch spec gets shell-guard denies of `git push` and `gh` for this session.
  D7 also states the residual plainly as a bound: the guard rows are a guardrail and not a barrier, and the bound says what catches each escape.
  It also says outright that a session approval from a non-author account counts.
  Stating only the residual, the option first applied, left that approval gap open, as fix-proposal finding 2 showed.
  A D7 criterion names the guard and the bound.
- F5 (MEDIUM): fixed differently.
  D11's fourth bullet routes the landing through Finalize's already-landed path, as proposed.
  It keeps the fast-forward of the local parent pair before that path, because `fabricengine.ParentHoldsTaskWork` (`internal/fabricengine/replay.go`) reads the local parent; dropping the fast-forward would squash the work again onto a stale parent.
  D11's bound now states that this path writes no merge record or conclude commit (`internal/fabricengine/merge.go` already-up-to-date probe), and the D11 criterion was updated.
- F6 (MEDIUM): the D6 record now holds the session's strand GUID, session id and engine.
  In D7, shuttle gains a resume of an opened session by its session id, and the row reads the identifiers from the record.
  The D7 criterion covers a re-entry after a crash resuming the recorded session.
- F7 (MEDIUM): on merged, D9 now stops a live session, then re-reads the head and tree.
  A dirty tree on the merged head is a halt naming the uncommitted files.
  Its way forward is discarding them, after filing any worth keeping as a board note, then `lym loom resume`.
  Committing them was dropped as a way forward, since it only leads to the lacking-commits halt (self-check).
  The D9 criterion was updated.
- F8 (MEDIUM): D11's refused-merge halt now gives a way forward per refusal.
  A disallowed merge method names `lym config landing --set squash=<true|false>` in the prime worktree (a `CategoryCrossPair` key), or allowing the method on the repository, then `lym loom resume`.
- F9 (LOW): D11's no now closes a recorded open pull request with the resolver's reason and removes the record, as D3's no does.
  D12's landed-run sentence now holds on both paths.
  The `PATTERN-github-auth` text now names PR-Review, Finalize and pair teardown as the only packages that open, merge or close a pull request.
  It no longer says which of them does which: the gate caps a PATTERN rule text at 280 characters, and D3, D11 and D12 state who does what.
- F10 (LOW): D9's approval bullet resumes the session only when the row stopped a live one for that read.
- F11 (LOW): D4's way forward is split into its two alternatives, and only the first ends in `lym loom resume`.
- F12 (LOW): D12 states the bound for a gone task worktree: the pull request stays open and a Warn names the task branch.
  The D12 criterion and the support log's Q7 match.
- F13 (LOW): the lacking-commits halt now says "filing them as a board note".

## Self-check

- fix-consistency, MEDIUM, `PATTERN-github-auth` text did not name Finalize closing on a no: fixed differently. The rule text now names the three packages that may open, merge or close a pull request, without saying which does which, since the gate's `pattern-change-over-cap` finding refused the first wording at 318 characters, over the 280 cap.
- fix-consistency, LOW, support log Q7 stale against D12's gone-worktree bound: fixed as proposed.
- fix-consistency, LOW, Scope's In list missing the quiet-check read: fixed as proposed.
- fix-claims, MEDIUM, D11 claimed the records side and its correspondence are handled on the already-landed path: fixed as proposed; the claim was dropped and D11's bound now states that no merge record or conclude commit is written.
- fix-claims, MEDIUM, the `squash` way forward did not name the prime worktree: fixed as proposed.
- fix-proposal, MEDIUM, F5 dropped the local parent fast-forward the already-landed path relies on: fixed as proposed.
- fix-proposal, MEDIUM, F3's residual bound let the session's approval stand in for the operator's: fixed with the launch-spec shell-guard deny, and the residual is stated.
- fix-proposal, MEDIUM, the D7 criterion claimed gating "before anything lands it" despite a GitHub merge: fixed as proposed; the criterion now says "lands through lym" and "is followed by" the halt.
- fix-proposal MEDIUM and fix-way-forward MEDIUM (duplicates), the ungated-merge halt's cancel clears a landed entry: fixed differently.
  The way forward is now `lym loom goto Finalize`, which takes the already-landed path and marks the task done.
  The lacking-commits bullet now comes first, so the halt only fires when the parent holds the task head.
- fix-proposal LOW and fix-way-forward LOW (duplicates), committing the dirty files only leads to another halt: fixed as proposed; the way forward is discarding them, after a board note.

Every fork returned a report.

## Disputed

None.

## Deferred

None.
