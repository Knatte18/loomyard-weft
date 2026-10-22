# Fixer report — discussion, round 1

## Changes

- F1 (BLOCKING), fixed as proposed: D3 gains a fourth condition, a clean task worktree read through a told seam, where a failed status read counts as dirty.
  The Bound names the HEAD and clean-tree conditions together.
  The writer-spawning criterion lists a dirty task worktree and a failed status read.
  The D3 heading, the Scope In line, the Constraints `PATTERN-told-geometry` entry, the reroute criterion and support log Q12 now carry the condition too.
  The Bound's old claim "as a gate-exhausted halt does" is dropped, since `Commit` commits only the landing directory (`internal/loomcli/darnwiring.go:166-170`).
- F2 (MEDIUM), fixed differently: D1 states that the classifier reads every failure identity of the log the record names, not only the record's tests.
  The proposed bullet would have made every package-level failure owned.
  I did not keep it: a package identity carries its import path (`internal/websterengine/verifyparse.go`), so a crash or timeout in a package outside the set would route to Darn-Write, the loop the Goal removes.
  D1 instead owns a package-level failure whose package is in the impacted set, and an opaque failure or a record with no failing test.
  New Acceptance criteria cover the reviewer's mixed case, an opaque failure, and a package-level failure outside the set.
  Support log Q3 follows.
- F3 (MEDIUM), fixed as proposed: D1's Rationale, the Goal and D2 now carry the replacement text.
  D2's reason flags packages whose tests import `internal/testkit/lyxbin`, and a matching Acceptance criterion was added.
- F4 (LOW), fixed as proposed: D3's tip bullet reads only the tips Merge-In would merge, the local branch alone under push skip.
  The writer-spawning criterion gains "an upstream tip outside HEAD on a run that skips pushes".
  Support log Q11 follows.
- F5 (LOW), fixed differently in wording: the Stuck-pair sentence now names "ahead of" and the reroute cap.
  I did not write "three reroutes", because that pins `rerouteCap`'s value.
  The count window now reads "since its latest Done or reroute-cap halt", per `rerouteCount`.
- F6 (LOW), fixed as proposed: D3 states that the gate-exhausted halt's way forward names the re-spawn or the Merge-In reroute.
  A matching Acceptance criterion was added.
- F7 (LOW), fixed as proposed: D4's last sentence names each gated producer as told the path, writing the file on an exhausted gate and removing it on a passing call.
- F8 (LOW), fixed as proposed: D6 and Scope name the `TestRows_PatternsDenyAndAllow` case, with the rows unchanged.
  D6 and its criterion use `github.com/Knatte18/loomyard/internal/x`.
  The support log's Turn 2 and Q2 follow.
- F9 (LOW), fixed as proposed: the Scope Out departure sentence, D3's 2026-10-20 sentence and the two restating Bound sentences are deleted.
  The 2026-10-20 sentence moved to the support log's Turn 2.
  Turn 3's and Q10's references to the moved text follow.
  The Auto-mode assumption for item 1 now holds the fact that the re-scope's landed list does not name `98b171b05`, and no longer points at Scope.
- The support log's Review rounds section records round 1.

## Self-check

- fix-consistency, LOW, Open risks names only the committed twin of the non-rerouted case: fixed as proposed, with a dirty-tree Open risk and its way forward.
- fix-consistency, LOW, Turn 2 still said the findings file lasts until the row's next Done: fixed as proposed.
- fix-consistency, LOW, Turn 2 still said "pinning row": fixed as proposed.
- fix-consistency, LOW, the Auto-mode assumption's "(see Scope)" led nowhere: fixed as proposed (merged with fix-proposal's LOW).
- fix-consistency, LOW, `PATTERN-fabric-git` named only the tip read: fixed as proposed, tip and status reads.
- fix-claims, LOW, the reroute-cap window omitted the latest cap halt: fixed as proposed.
- fix-way-forward, LOW, the cap halt's resume only reroutes back to the Stuck Merge-In: fixed with my own change.
  The reroute's reason now says the parent moved and Merge-In takes it in.
  The Bound says the cap halt embeds that reason, which points the parent at Merge-In's pair repair before the resume.
- fix-way-forward, LOW, the dirty tree with a fixed parent loops with no named exit: fixed as proposed (merged with fix-consistency's first LOW).
- fix-proposal, MEDIUM, F2's proposal owns every package-level failure, wherever its package lies: fixed with my own change (see F2 above).
- fix-proposal, LOW, D3's status seam is not tied to Merge-In's dirty reader: fixed as proposed, the seam is over the dirty-paths reader Merge-In's clean check uses.
- fix-proposal, LOW, "(see Scope)" now points at a bare closure: fixed, merged above.

## Disputed

None.

## Deferred

None.
