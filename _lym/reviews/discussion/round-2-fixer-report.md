# Fixer report — discussion round 2

The `decision`-class findings F12, F13, F14 and F15 went to the parent, and no ruling on them reached this round.

## Changes

- F1, fixed as proposed: the adopter finishes a landing past its parent-side merge.
  D1's act rule puts `hand_over` ahead of the landing `act`, and that `act` no longer covers a run another machine owns.
  D4 hands such a run over, D6 step 4 and D9 drop their carve-outs, D7 sends the retired callers' skip to the owner,
  and the `PATTERN-owner-machine` constraint and its declared change say the owner finishes it.
  D1 and D6 criteria are reworded, and a D4 criterion is added.
- F2, fixed as proposed, with one addition: D2 pushes the records side first and the code side once the records side lands.
  D4's hand-over pushes no code side for the rejected transition.
  D4 states how the adopter's code branch is read: each records commit names its code tip, the act rule waits until origin holds that tip,
  and a tip still missing after a new code-tip grace is given up by returning the run to its previous transition.
  The grace is listed among the named constants and auto-mode assumptions.
  The `PATTERN-pair-remote` constraint and its declared change follow, and D2 and D4 criteria are added.
- F3, fixed differently: the review proposed a new mode of `internal/pairteardown`, but its `Request.Local` mode already removes a pair with no origin step and no board write (`internal/pairteardown/teardown.go:54-56`, `:419`).
  D6 step 4 and the `PATTERN-pair-remote` constraint name that mode in place of fabric's nonexistent owned-elsewhere removal.
- F4, fixed as proposed: D3 gives a finished cancel the shed state `cancelled`, which `battencli.ClearDoneRun` never takes.
  `lym batten cleanup` removes such a record once its pair is gone, and otherwise refuses it naming `lym fabric remove <slug>`.
  D3 says a landed run's record goes at the done-run clear.
  D1's `landed` covers a slug with no batten record and a board `done`.
  D3 and D1 criteria are added.
- F5, fixed as proposed: the upgrade posts one notice naming the hold, its machine and a re-run of `lym adopt` there, and gets a criterion.
- F6, fixed as proposed: D10 adds a trailing mirror, the bound covers a takeover before it lands, and a criterion is added.
- F7, fixed as proposed: the session record names the orch's last launcher or resumer.
  A mirror that loses its lease to that machine's mirror writes nothing, and the restore follows the session record.
  The D10 bound and criteria follow.
- F8, fixed as proposed and narrowed after the self-check: see the Self-check section.
  D2 takes a gate admission before the commit and releases it when the push lands or turns owed, and a criterion is added.
- F9, fixed as proposed: the takeover retry in D4 re-reads the hold and owner from a fetched Bolt and stops, naming the release, and a criterion is added.
- F10, fixed as proposed: D8 step 3 fast-forwards the moved runs' local branches once the takeover lands, and a criterion is added.
- F16, fixed as proposed: D11's condition names the FSM state, the pause and hold coverage and the owner, not the act rule's answer.
- F17, fixed differently for D5: D3's owner refusal names the same verb on the owner machine, or `lym adopt` here first, as proposed.
  The review proposed `lym machine name <other>` on this machine as D5's way forward.
  A rename adds a former name and drops none (D5's third bullet and `internal/machinecli/set.go`), so the clash would stand.
  D5 adds `lym machine name --forget-former <name>` (new), which drops the clashing former name, and names it as the way forward.
  The `PATTERN-refusal-way-forward` line covers D3's and D5's refusals, and D3 and D5 criteria are added.
- F18, fixed as proposed: D7 makes every verb that asks the act rule fetch Bolt first when origin answers, rewords the bound, and adds a criterion.
- F19, fixed as proposed and narrowed after the self-check: see the Self-check section.
- F20, fixed as proposed: Q5 lists a hand pass in place of a rejected push.

## Self-check

Every lens returned a report: fix-consistency, fix-claims, fix-proposal and fix-way-forward.
Duplicates across lenses are merged on one line.

- fix-claims 1 (MEDIUM), with fix-proposal 4 (MEDIUM), on `Admit`: fixed with my own change.
  D2 takes the admission before the commit and releases it once every side lands or the push turns owed, and an owed push's retries wait at the gate.
  This closes the waiting-before-admission gap and the in-flight marker held through an offline retry.
- fix-claims 2 (MEDIUM), with fix-way-forward 2 (MEDIUM) and fix-consistency 3 (LOW), on `--forget-former`: fixed.
  D5 declares the flag as new and says what it does to owners recorded under the forgotten name.
  D8 step 1 names D5's refusal and its way forward, and a D8 criterion is added.
- fix-consistency 1 (MEDIUM), D8's bound against the code-tip push: fixed, the bound names the code tip of a transition already on origin.
- fix-consistency 2 (MEDIUM), with fix-way-forward 1 (MEDIUM) and fix-proposal 2 (MEDIUM), on the code-tip wait: fixed with my own change.
  The owner's dobby fast-forwards to the tip, a tip still missing after the code-tip grace is given up by returning the run to its previous transition,
  and a machine pushes an owed tip only while origin's last transition names it.
  D8 step 3 names the dobby, and the D4 criterion covers the give-up.
- fix-consistency 4 (LOW), stale Turn 5 lines in the support log: fixed with my own change.
  The Turn 5 lines record round 1 as it happened, so a Round 2 entry under Review rounds records the reversals instead of rewriting them.
- fix-consistency 5 (LOW), D1's teardown sentence: fixed, it points to the states D11 names.
- fix-way-forward 3 (MEDIUM), with fix-proposal 3 (MEDIUM), on several machines' upgrade holds: fixed.
  A first read that finds another machine's hold writes none, and the notice names the subjects to re-adopt.
- fix-way-forward 4 (MEDIUM), with fix-proposal 6 (MEDIUM), on `BeamUp` after an early release: fixed.
  Only an adopt released after its step 3 landed starts dobbies, and an earlier stop names a re-run of `lym adopt`.
- fix-way-forward 5 (LOW), restore with no mirror: fixed, a local transcript is resumed when no mirror stands.
- fix-proposal 1 (MEDIUM), the old owner's Finalize writing the parent side: fixed.
  D4 makes the parent-side merge a transition on origin, and has each later Finalize push read the owner from a fetched Bolt first.
  A D4 criterion is added.
- fix-proposal 5 (MEDIUM), landed runs paused by the upgrade: fixed, only a run in a state that is not terminal becomes a pause intent.
  The D7 criterion covers the drop.
- fix-proposal 7 (MEDIUM), the old owner's orch start rewriting the session record: fixed.
  An orch start writes the record only after a fetched Bolt names this machine as owner, and a D10 criterion is added.

## Disputed

None.

## Deferred

- F11: the research needs `claude --resume <id>` run on a transcript copied into another worktree's project directory.
  Every headless Claude launch is refused in lym sessions, and an interactive resume needs an operator pane, so I cannot run it this round.
  Its result, with the Claude Code version, goes under D10's Facts.
  If the session does not resume with its history and id, D10's restore has to change to whatever the result allows.

## Rulings

None reached this round.
