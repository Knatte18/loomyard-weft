---
verdict: APPROVED
---

No defect found in the review range (`git diff $(git merge-base main HEAD)..HEAD`, no rework round exists).
The range was read against the plan's 25 cards, `PATTERN-owner-machine` in its declared text, and the focus directive's seven questions.

## Focus verdicts

- Owner check coverage: batten start in `armAt`, batten resume through `ownerPreflight` and `reconcileResume`, loom start and resume in `refuseOwnedElsewhere`, `InnerRun`'s `ownerStep`, Finalize's `ownerChecks`, the wake's `holdRuns` and orch start's `pauseOwnedRuns` each judge the owner before they act; no path writes or claims without it.
- Owner write rule: `boardengine.SetOwners` is the only writer of `owner_machine` and `prime_owner`, besides the replay's origin-wins merge; `checkOwnerPayload` refuses an owner in an upsert payload; only `adoptcli/adopt.go` sets `Takeover`.
- Effective owner resolution: `Owners.Effective` applies each case as the entry states it.
- Finalize carve-out: `ownerChecks` reads `ParentHoldsTaskWork` first and a parent holding the work skips both owner checks and the unreachable halt, which is the line the PATTERN entry draws.
- Declared PATTERN change: `pattern/PATTERN-owner-machine.md` opens with the declared text, the overview line is shortened, the other lines drop only filler words, and card 25's commit carries the `Pattern-Departure:` trailer naming the line cap and the headroom line.
- Machine name: `internal/machinename/machinename.go` holds the only production `os.Hostname`, and a sole-source scan pins it.
- Test economy: each added behaviour has a pinning test, and no new test duplicates an existing one.

## Rejected

- Fork `code-affected-set`, "`adopt` may be missing from the `ly:cli` index": the skill renders `lyx help index` from the running binary, so there is no list to update.
- Fork `code-tests`, "`claimBoardEntry` untested": `TestWire_CreateWorktreeClaimsTheBoardEntry` in `internal/battencli/wire_seams_test.go` covers the setter error, the lost write and the real setter.
- Fork `code-tests`, "`adopterFirstRepair` untested": `internal/landingshed/finalize_test.go` rows pin the reset note, the rebase note and the no-repair cases.
- Fork `code-tests`, overlap of the loom unit and integration owner tests: they reach different layers, and the fork found no duplicate.
- Fork `code-correctness`, NIT on `writeOwners` re-saving a prime that already equals the machine: the production callers read `Prime != ""` first, so the case is unreached and the result is not wrong.
- Forks `code-affected-set` and `code-correctness`, `git reset --keep` and `git rebase --abort` in the repair note: `PATTERN-no-denied-recovery` lists them as not denied (`--keep`) and outside its closed list.
