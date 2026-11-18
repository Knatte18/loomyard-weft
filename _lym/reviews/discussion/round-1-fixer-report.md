---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F11
    disposition: fixed-differently
    reason: The proposal finds a removed hub's server by its name's readable half lying under the root, but reedengine.ServerName's readable half is only filepath.Base of the hub path, which cannot place a server under the root; D5 instead keeps a hub list in the root from which the reaps derive every recorded hub's key, whether or not its directory still exists.
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
---

# Round 1 fixer report: Hardener decision record

F10 is a `decision`-class finding held by the parent, so this report leaves it out.

## Changes

- F1: D8 now states that `agentconfig.Load` refuses an unknown role key, pinned to `skillLists` and to the test rows `UnknownRole` and `PairWithUnknownRole`.
  It ships a `configreg.Reinterpretation` on the `agents` module that strips the treadle role keys from `skills` and `efforts`, applied by `hubreconcile`.
  Scope and the D8 criterion follow.
  The support log's Turn 3 bound-breaker line now records the original false fix and its correction.
- F2: D4 runs the hub keys `harden-live` and `harden-live-copies` through `lym gate test --gate <key>`, with `packages` as explicit targets.
  Both keys ship in the gate template with tiers `[0]`, and lym's hub sets them to `[2]`.
  `--copies` requires `--gate`.
  Steps 1 to 3, the bound, the rationale and the criteria now use the key form, and step 3 keeps the rule that no key names tier 3.
- F3: D5's reap ends recorded processes through `runproc.End` and uses the `/proc` kill only for unrecorded processes, named as a new orphan reaper.
  A new `## PATTERN changes` section admits that reaper to PATTERN-child-processes beside `lym tender reap`, bounded to the sandbox root.
  Criteria added.
  After the round's gate flagged the section's bullet as malformed, the bullet was rewritten as one line in the gate's
  `` - `PATTERN-<id>` — changes: <text> `` shape.
- F4: D4 states the slot rules from `internal/gateslot/doc.go`.
  The `live` gate holds no slot of its own, so each call acquires as a top-level run.
  `--copies n` acquires n slots, and an n above the pool's count is clamped and reported.
  The bound and the criterion follow.
- F5: "serially" is dropped from D4 step 1.
  The merge bar is restated as one green `harden-live` run, in the rationale and the criterion alike.
- F6: D3 expands the lists cyclically to the hard cap `hard_max_bounces`, which is added to D9 with a default of 10.
  D2's Preflight refuses a reviewer list with adjacent duplicates under that expansion, with the `run_settings` write as its way forward.
  The rotation criterion now covers rounds after a parent's continue, and a Preflight criterion is added.
- F7: D4 adds `live_attempts` (D9, default 2).
  When the attempts run out, the row ends with the live report red and its failures recorded as deferred findings, and the run is not halted.
  Criterion added.
- F8: D5 keeps one stray report per round, and each reap appends to it with the row and the reap point.
  D7 copies the archive after the fix row's end-of-row reap.
  A criterion now requires that a review-row stray appears in the archived report.
- F9: D10 refuses two more cases: a segmented build row that no edge from its segment's Bouncer reaches, and one whose segment has no Bouncer.
  The bound and the criterion are restated to match.
- F11 (fixed differently; see the frontmatter reason): D5 keeps a hub list in the root, and the reaps derive server keys from it.
  A hub made outside the root is out of scope, and the rubric forbids making one.
  Finalize and a halted campaign run the reap.
  The reason cited is fabric's `.git` refusal.
- F12: D3 states that the segment-budget projection, including `loomCLI.reviewBudget`, reads Harden-Review's cap from `hardener.yaml`.
  A criterion for `lym loom status` is added.
- F13: D3's verify gate runs `harden-burler-fix`, narrowed to the touched trees when the diff lies wholly under `tree_scope`, as darn does (`darnTreeScope`).
  D9's `tree_scope` entry and the D3 criterion are aligned with it.
- F14: D3's rationale now cites `internal/burlerengine/models.go` `pickRound` behind `RoundModels.Pick`.
  A new Constraints bullet cites `internal/battencli/wire.go` `childRecipeSupported`.

## Self-check

- fix-consistency, LOW: the support log's Turn 3 rotation line was stale. Fixed as proposed: the line now notes the move to the hard cap.
- fix-consistency, LOW: D6 did not read the gate-written deferrals. Fixed as proposed: D6's facts rendering now lists the deferrals the fixer leaves and the ones the `live` gate records.
- fix-consistency, LOW: the halted-campaign reap was worded two ways. Fixed as proposed: the criterion now says "as it halts", as D5 does.
- fix-claims, MEDIUM: the `.git` "at any depth" claim did not hold for both cleaners. Fixed as proposed: D5 now cites `ignoredTaskPathDeletable` and notes that the root lies in the worktree's ignored `.lym`.
- fix-claims, LOW: `gateslot.AcquireSlots` is a method. Fixed as proposed: the citation is now `gateslot.Pool.AcquireSlots`.
- fix-proposal, MEDIUM: a hub that leaves the live keys at the template's `[0]` gets a green gate that drove nothing. Fixed with own change: Preflight now refuses a run whose hub's own `gate.yaml` does not set the key, instead of refusing "no tier above 0" as proposed, because a hub's live tier need not be above tier 0.
- fix-proposal, MEDIUM: `max_bounces` above the hard cap brings the clamp back. Fixed: Preflight refuses it, with the `run_settings` write as its way forward, and a criterion is added.
- fix-proposal, MEDIUM: the next round's `verify` could halt on a deferred live failure, and the gate-written deferrals had no writer. Fixed: lym's hub sets `harden-burler-fix` to the tiers below its live tier, and the live report records the deferred findings, which D6 reads.
- fix-proposal, LOW: a runproc record-format change made by a round was unbounded. Fixed as proposed: D5 now states the bound.
- fix-way-forward, MEDIUM: the gate-time tier-3 refusal could not be acted on mid-campaign. Fixed as proposed: the check moved to Preflight (D2), and the criterion moved with it.
- fix-way-forward, MEDIUM: a red gate left by deferred failures made CONVERGED unreachable. Fixed: D6 now states that such a campaign ends capped by design and its parent decides from the facts, and the criterion says so.
- fix-way-forward, MEDIUM: D10's "drop the segment" way forward did not make progress. Fixed as proposed: the way forward now names only adding the Bouncer row.

## Disputed

None.

## Deferred

None.

## Rulings

None.
