# Fixer report — discussion round 1

All 18 findings are fixed in `_lym/discussion/decision-record.md`.
The support log is unchanged.

## Changes

- F1, fixed as proposed: D4 bullet 2 now fetches and reads origin's tip before any replay, and hands over without replaying.
  Bullet 3 states that a rejection with no takeover entry replays as today, which stands in for the proposed "Only when it holds none" sentence, so the rule is said once.
  Bullet 4 has the adopter rewrite its takeover entry onto origin's tip, never replaying.
  D2's rejected-push bullet and the D4 criteria follow, and a new criterion covers the no-replay hand-over.
- F2, fixed as proposed: D7 writes `hold` slug entries, no hub entry and none for the fix task.
  A new bullet exempts the orch, adopt's process and its deploy and check children, and the operator's cancel.
  D8's Bound states what goes on.
  A D7 criterion covers the fix task's gate and the orch.
- F3, fixed differently in one part: `--release` clears any hold from any machine, D7's and D8's Bounds name the stopped-adopter case, and a D8 criterion covers it.
  The proposal's "no way forward an agent reads" for D8 step 1's refusal was not taken,
  because the Constraints' `PATTERN-refusal-way-forward` line requires every refusal the holds add to name its way forward.
  The refusal instead names waiting for that machine's adopt, or asking the operator when that machine has stopped, and never names `--release`.
- F4, fixed differently in placement: the landing carve-out answers `act` under a hold and when another machine owns the run, and D7 and the D1 criterion say so.
  The proposal listed it before every `wait`.
  It sits after the owed-push `wait` and yields to a pause, because D2's Bound says no run acts while origin is unreachable,
  and a hub entry is the one entry that holds a parent-lock holder (`internal/pauserecord/doc.go:52`).
- F5, fixed as proposed: a D7 bullet keeps `hold` entries apart from the operator's and hibernate's hub entries,
  including the one-entry-per-scope collision and `lym tender resume`.
- F6, fixed as proposed: a cancel keeps the batten record and writes its finish.
  D1 reads `cancelled` and `cancelling` from it.
  D11 converts a kept pair from before the upgrade into a batten record with a finished keep.
- F7, fixed as proposed: D3 states what each slug verb does offline and its bound, and adds an offline-pause criterion.
- F8, fixed as proposed: D3's owner refusal reads a Bolt the verb fetches.
  The act rule answers `wait` for `paused` and `cancelling`, and `lym tender resume <slug>` and `lym batten cancel` set their own run's intents aside.
- F9, fixed as proposed: D2 retires every detached push that carries a run transition, the teardown-lock respawn included.
  A transition waits for a held teardown lock.
  The anchor change's push and tender's prime push stay detached, each with its reason.
- F10, fixed differently: the proposal had a lost lease write nothing until the next orch start.
  That keeps the old owner's late mirror on origin, so the adopter's next restore would take it.
  A lost lease now re-reads the ref and pushes again while the act rule does not answer `hand_over`.
  The old owner's mirror stops at `hand_over`, and D10's Bound states the window and that the old owner's turns after the takeover do not travel.
  The D10 criterion follows.
- F11, fixed as proposed: `gh repo view Knatte18/loomyard-weft --json visibility` answered `PUBLIC` on 2026-11-15.
  That fact is recorded in D10's Facts, and the Open risk is narrowed to the operator's choice.
- F12, fixed as proposed in D7's Bound.
- F13, fixed as proposed in D8 step 1.
- F14, fixed differently: dropping a released task's entry let a queued task start under an adopt-hold from before the upgrade.
  The self-check (fix-proposal F14) showed this.
  The first read after the upgrade now also writes a Bolt `adopt` hold, and the task stays `queued` under it.
- F15, fixed with the proposal's second option: D9 states that the first orch start after the upgrade refuses over the pre-upgrade orch, and the operator exits it once.
- F16, fixed differently: the try count lives in the `adopt-fix` hold, as proposed, but per subject.
  The limit is per subject (`internal/adoptcli/fixloop.go:356`).
- F17, fixed as proposed: the `claude-account-pick-simple` Open risk is deleted.
- F18, fixed as proposed: the question-ledger assumption is deleted.

## Self-check

- fix-consistency, MEDIUM, landing carve-out skips the owed-push and pause waits: fixed differently.
  The landing bullet now sits after the owed-push `wait` and yields to a pause, and the D1 criterion follows.
  This is merged with fix-proposal F4.
- fix-consistency, MEDIUM, D7 criterion "fix task alone gets passes" contradicts the landing exemption: fixed as proposed.
- fix-consistency, MEDIUM, the pause deferral's reason for retiring is false while two detached pushes stay: fixed with own change.
  The deferral now no longer applies to a transition's push, and it stays for the detached ones.
- fix-consistency, LOW, the Open risk on the orch under a hold is stale: fixed as proposed.
- fix-consistency, LOW, and fix-way-forward, MEDIUM, waiting is a dead end when the adopter stopped: fixed as proposed.
  The refusal adds asking the operator when that machine has stopped.
- fix-way-forward, MEDIUM, no process owns the offline cancel's finish: fixed as proposed.
  The cancel's output names a re-run of `lym batten cancel <slug>` once origin answers.
- fix-way-forward, MEDIUM, `lym tender resume <slug>` under a hold names no way forward: fixed as proposed.
  It refuses, naming the hold's machine, waiting, or the operator.
- fix-way-forward, LOW, the kept-pair conversion catches a claimed pair that stopped before its batten record: fixed as proposed.
  The conversion requires an unclaimed board entry.
- fix-claims: no findings.
- fix-proposal, BLOCKING, F4 carve-out ahead of the owed-push wait: fixed.
  This is merged with the fix-consistency finding above.
- fix-proposal, MEDIUM, F3 `--release` over a live adopt leaves two adopters: fixed with own change.
  An adopt re-reads its hold before each step from step 2 on and stops once it is gone.
- fix-proposal, MEDIUM, F6 a kept finished-cancel record blocks a re-released slug: fixed with own change.
  The batten record goes with the pair's teardown.
- fix-proposal, MEDIUM, F7 an offline intent is silently dropped by a hand-over: fixed with own change.
  D3's Bound and D4's notice name each dropped intent.
- fix-proposal, MEDIUM, F14 dropping queued entries lets tasks start: fixed with own change.
  The upgrade writes a Bolt `adopt` hold.
- fix-proposal, LOW, F16 one try count for several subjects: fixed with own change.
  The count is kept per subject.

## Disputed

None.

## Deferred

None.

## Rulings

None reached this round.
