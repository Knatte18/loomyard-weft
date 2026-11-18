---
dispositions:
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed
  - id: F17
    disposition: fixed
  - id: F18
    disposition: fixed
  - id: F19
    disposition: fixed
---

# Round 2 fixer report

Every finding's premise held against the code:
- `reviewbudget.Categories` makes auto-accept Guard.
- loom's fixer and split fans are Guard, and so is darn's `tree_scope`.
- `configengine`'s local rule admits only CategoryNone keys.
- `acquireSlot` and `exe.inherited` behave as the review says, at `internal/gatecli/run.go`.
- PATTERN-test-speed's `{test_targets}` narrowing holds.
- `internal/pairteardown` holds no sandbox reap.

## Changes

- F15:
  - D9 replaces "may override any of them" with a classification per PATTERN-config-layers that cites the four precedents.
    `auto_accept`, `reviewer_fan`, `fixer_fan` and `tree_scope` are Guard and set only in the hub's file; every other key is CategoryNone.
  - D9 names the keys `run_settings` may set, and a Guard key is refused at seed through `HubOnlyWayForward`.
  - D2 states the bounds of `packages` and `llm_driving`, and its rationale now says `run_settings` carry CategoryNone keys.
  - D6 bounds `auto_accept`: a Harden-Bouncer never auto-accepts a round whose live report is red, and escalates it to the parent instead.
  - Criteria added: a Guard key in `run_settings` is refused, `packages` and `llm_driving` load from `run_settings`, and the auto-accept bound holds on a red report.
- F16:
  - D2 adds a Preflight refusal for packages holding no test of the live key's tiers, with both ways forward.
  - D4 has the live report record the binaries each run started; a run that started none reads red.
  - Two criteria added.
- F17:
  - The Open risks treadle question is deleted.
  - D8's rationale quotes the 2026-10-16 note as written and cites the 2026-11-18 operator entry that replaces it.
  - Support-log Q13 is resolved, citing that entry.
  - Neither added entry was edited.
- F18:
  - D4's `--copies` paragraph cites `acquireSlot`, `gateslot.Pool.AcquireSlots` and the `LYM_GATE_SLOT` / `exe.inherited` path.
  - It decides that the call's lease replaces its tier runs' own acquires.
  - The copies are runs of the key's rendered tier command, each under its counter, timeout, watchdog and memory guard.
  - The bound now says a call holding every slot drains the pool for the length of the call.
  - The criterion now asserts that a call whose lease equals the pool completes and runs n copies.
- F19:
  - D5's teardown bound has `internal/pairteardown` spawn `lym harden sandbox clear` in a worktree whose sandbox root exists before it removes the worktree.
  - It names every route: Finalize, the capped or circling halt, `lym fabric remove` and Worktree-Teardown.
  - The Finalize criterion is extended to those routes.

Changes from the self-check, in the same files:
- `clear` is defined as running the full reap, process ends included.
  `clear` also runs in any worktree whose sandbox root exists.
  When the clear fails, pairteardown keeps the worktree and names a way forward.
- The Preflight live-tier refusal is per package and also covers `harden-live-copies` when copies will run.
- The live report's mark is `harden-live`'s outcome alone; a copies run that started none is recorded without failing the gate.
- `--copies` leases n times the tier's slot count and is refused for a key naming more than one tier.
- Scope In names the pairteardown clear and the auto-accept bound.
- Support-log Turn 3's treadle line notes the later settlement.

## Self-check

- fix-consistency, MEDIUM: the pairteardown route ran `clear`, which D5 defined as narrower than the reap. Fixed: `clear` runs the reap.
- fix-consistency, LOW: Scope In did not name the pairteardown clear or the auto-accept bound. Fixed: a Scope bullet was added.
- fix-consistency, LOW: support-log Turn 3 still described an open treadle question. Fixed: the line notes the settlement.
- fix-way-forward, MEDIUM: duplicate of the fix-consistency `clear` finding. Fixed by the same change.
- fix-claims: no findings.
- fix-proposal, MEDIUM: a copies run with no tests read red with no rule for how it affects convergence, and Preflight did not check the copies key. Fixed: the report's mark is `harden-live`'s alone, and Preflight checks the copies key's tiers.
- fix-proposal, MEDIUM: the Preflight refusal was list-wide, so a mixed list left a package unverified. Fixed: the refusal is per package.
- fix-proposal, MEDIUM: the arbiter added entry still prescribes D8 rationale text that points at the removed Open risks question. Deferred, see below.
- fix-proposal, LOW: the n-slot lease ignored the tier's own slot count and keys with several tiers. Fixed: the lease is n times the tier's slots, and a multi-tier key is refused.
- fix-proposal, LOW: nothing said what teardown does when `clear` refuses or fails. Fixed: `clear` runs wherever a sandbox root exists, and on failure pairteardown keeps the worktree with a way forward.

## Disputed

None.

## Deferred

- Self-check fix-proposal finding 3 (arbiter entry conflict).
  The added entry "D8's rationale quotes the 2026-10-16 treadle note as written" still prescribes D8 rationale text that calls deletion "the open parent question in Open risks".
  The later operator entry settled that question.
  F17's fix, as the review directs, leaves both added entries untouched.
  Resolving the conflict needs `lym loom decision replace` from the parent or the operator, which is outside this fixer's write surface.
  This report is the only flag: the instructions allow a parent message only for rulings.

## Rulings

None.
