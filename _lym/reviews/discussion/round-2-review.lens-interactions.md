```yaml
lens: interactions
outcome: reported
```

Interactions lens over `decision-record.md` D1 to D10: pairs that act on the same state, run with each other in force.

**Pairs checked**
- D1 records-first order × D1's anchor-change push kept detached × D7's lift paying deferred pushes: **broken** (F1)
- D1 pending code-side flag left on disk × D2 hand-over that pushes nothing more: **broken**, same path as F1
- D1 owed push × D5's exemption from `bolt-hold` entries: **broken** when the push turns owed after the entry stands (F3)
- D1 owed push × D4 step 4, which archives runs owned elsewhere: **broken** (F2)
- D1 network-gate admission × D9 pause and idle confirmation: held
- D1 owed-push `PushOwed` rename × D4 retiring the origin-owed marker: held
- D1 teardown-lock wait turning owed × D7 teardowns: held
- D2 takeover entry × D3 registry comparisons: **broken** (F4)
- D2 hand-over mark × D6 step 3 dropping it × D7 mark-driven removal: held
- D2 Finalize owner read × D6 order (owner in step 1, takeover in step 3): held
- D2 Finalize cut before parent push × D5 hold × act rule `PastParentMerge`: held
- D2 code-tip give-up and lease × D1 records-first owner push: held
- D2 adopter's rejected takeover push × D6 step 1 hold re-read: held
- D4 step 4 removal × D7 orphan and mark teardowns: held
- D4 dobby-start catch-up × D9 stop and D10 restart: held
- D3 registry × D6 step 1's single commit (hold, owner, registry): held
- D3 registry × D10 `OwnedWorktrees` filter: held
- D3 lost `machine.json` bound × D6 own-hold re-run: **broken** in a narrow case (F7)
- D5 upgrade hold × D6 own-hold re-run from step 2: held
- D5 upgrade hold re-run × D6 step 2 archive and reset: **broken** (F5)
- D5 `bolt-hold` entries × D6 step 8 drop × D9 own-entry lift: held
- D5 `adopt-fix` fix task × the `Held` input: held
- D5 changing `runfsm`'s verdict × Scope's "never redesigns `runfsm`": **broken** (F6)
- D7 kept-cancel record written at upgrade × D1 synchronous path under a hold: held
- Criterion "after a takeover, the old owner leaves origin's code branch unchanged" × D2 hand-over pushing the owed code tip: **broken** (F8)
- Remaining criteria and constraints against the decisions they touch: held

**Findings**

**F1. BLOCKING, design.** Location: D1, the anchor-change push "stays detached", and D7 "still pays a detached push a pause deferred".
- The detached pair push that stays sends the code side before the records side, and it does no takeover read.
- `internal/fabricengine/anchorchange.go:557`: the anchor change's Finish calls `SpawnStatusPush(pair, "")`.
- `internal/fabricengine/coalesce.go:153-164`: the detached push behind it pushes the warp (code) side before the weft (records) side.
- `internal/tenderengine/resume.go:250-251`: a lift spawns `deps.PushPair(slug)` for each pair a lifted entry covered.
- So a task pair's code side can reach origin before its records side, and after another machine's takeover entry has landed.
- In the same push, the code-side pending flag D1 leaves "for its worktree's next push" is taken (`pendingflags.go:168-173`) and its code is pushed first.
- This breaks D1's claim that "the order holds for every caller", the D1 criterion that such a code side "never reaches origin before its records side, nor after a takeover entry", and the declared `PATTERN-pair-remote` text "after the records side".

**F2. MEDIUM, design.** Location: D4 step 4 against D5 and D1's owed-push guarantee.
- D6 step 1 pushes the Bolt owner change before step 3 pushes the takeover entries.
- An old owner whose catch-up runs in that window reads the run as owned elsewhere.
- D4 step 4 then archives the run's local tips, owed transitions included, and removes the pair.
- No takeover entry is on origin yet, so those owed commits could have landed.
- D5 exists so that "its owed push passes the gate and lands before an adopter's takeover reads origin", and D1's bound is that "an owed push holds the run at its transition until it lands".
- Step 4 says nothing about pushing owed records before it archives.
- Evidence: D4 steps 3 and 4 and D5's owed-push bullet in `decision-record.md`.

**F3. MEDIUM, design.** Location: D5, "A run whose push is owed gets no `bolt-hold` entry".
- Case: a machine reads a hold and writes a `bolt-hold` entry for a run whose transition push is still in its first attempt, so not yet owed.
- D1 lets that first attempt pass under its admission.
- If the first attempt fails, the push turns owed and its retries "wait at the gate like any network call".
- The `bolt-hold` entry already standing now blocks those retries, so the push cannot land before the takeover.
- D5 never says the entry is dropped when a push turns owed.
- Evidence: D1's admission and retry bullets; D5's owed-push bullet.

**F4. MEDIUM, design.** Location: D2's takeover-entry comparison against D3's "every comparison of a recorded machine name … goes through the registry".
- The takeover entry names a machine by name, and the Constraints run the takeover read in `internal/fabricengine`.
- `PATTERN-fabric-boundary` says fabric learns nothing of the board, and the registry lives in Bolt's `board.json`.
- Counterexample:
  1. Machine X adopts as `osl46`, so the takeover entry names `osl46`.
  2. X then runs `lym machine name osl-1046`. The sweep rewrites Bolt owners only, never takeover entries in run records.
  3. A later records push of X is rejected over another run's commits on a shared records branch.
  4. X reads a takeover entry naming `osl46`, which is not `osl-1046`, and finds no hand-over mark for it.
  5. X hands its own run over to itself.
- The record never states how D2's comparison reaches former names or the registry from inside fabric.
- Evidence: `internal/machinename/machinename.go:83` (`Names` gives the former names); D2 and D3 in the record.

**F5. MEDIUM, design.** Location: D6 step 2 against D5's upgrade hold and D4 step 3.
- D5's upgrade hold sends the operator to re-run `lym adopt` of subjects whose runs this machine already owns.
- Before the upgrade, a run's push was detached and could have lagged, so local commits may not be on origin.
- D6 step 2 "archives local tips origin lacks … resets them to origin's" for each moved run.
- That runs before step 4's catch-up, whose step 3 would have pushed those commits as owed.
- The machine's own transitions end up under archive tags and the run steps back.
- The record never says whether a run this machine already owns counts as a "moved run" in step 2.
- Evidence: D5's upgrade bullets; D6 steps 2 and 4; D4 step 3.

**F6. LOW, consistency.** Location: Scope "Out" against D1 and D5.
- Scope says a decision "never redesigns" `runfsm`, "bar D1's rename of one input and the wording of its refusal".
- D1 also adds the owed push's last error to `runfsm`'s inputs and verdict.
- D5 adds the hold's machine to `runfsm`'s inputs and verdict.
- The exception should name both.
- Evidence: `internal/runfsm/act.go:12-31` and `:61-74` carry no such fields today.

**F7. LOW, design.** Location: D3's bound for a lost `machine.json` against D6 step 1.
- A machine that lost `machine.json` while its own hold stood reads that hold as another machine's: the old name now belongs to its old registry entry.
- So it needs `lym adopt --take-hold`, which agents are denied.
- D3's bound says only that the machine "registers under a new name and adopts its own subjects again".
- Evidence: D3's bound; D6 step 1's refusal of a hold another machine wrote.

**F8. LOW, consistency.** Location: the D2 criterion "after a takeover, the old owner's next transition and its `lym loom commit-records` leave origin's code branch unchanged".
- D2's hand-over "pushes the code tip the run's last transition on origin names when origin's code branch lacks it".
- So when the old owner owes that tip, a hand-over does change origin's code branch, and the criterion as worded fails.
- Evidence: D2's hand-over bullets and its acceptance criteria.

**Tool calls**
1. `grep` of `internal/fabricengine/anchorchange.go` push calls and `internal/tenderengine/resume.go` `spawnOwed`. The manifest had neither the anchor change's push path nor what a lift spawns per pair, and F1 rests on both.
