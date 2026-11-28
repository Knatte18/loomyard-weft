---
dispositions:
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
---

# Round 2 fixer-report

Every change is in `_lym/discussion/decision-record.md`.

## Changes

- **F13** (fixed): D4's jacquard way forward now names two cases, as D5's `lym loom start` refusal does.
  A pair batten made keeps the batten sequence.
  Any other pair is re-seeded from its task worktree: `rm <seed path>`, `lym shed seed self --recipe tapestry --driver <driver> --param parent=<parent>`, `lym loom goto --to Discussion-Write`, `lym loom start`, then `lym loom resume`.
  The reviewer left this way forward for the fixer to establish in the code.
  Evidence read:
  - every loom verb reads its recipe from the seed (`internal/loomcli/arm.go` `resolveRecipe`);
  - `shedengine.Status` names no recipe (`internal/shedengine/status.go:187`);
  - goto moves a halted run back to an earlier row, and tapestry holds Discussion-Write before Record-Seed (`internal/loomcli/cli.go:498-516`);
  - a start over an existing status file commits the seed on every call (`internal/loomcli/sharedbootstrap.go:158-163`).

  That evidence is recorded as a new Fact under `## Constraints`.
  The D4 criterion now names one reason per case.
  A tier 1 criterion, added in the self-check, reaches the re-seed way forward.
- **F14** (fixed): the fact is settled.
  A motif run commits only on the records side (`internal/loomcli/wiring.go:506,575`, through `CommitAnchoredPaths`).
  Its task branch therefore holds no commit beyond its parent, and Remove deletes it locally and on origin (`remove.go:307,366`, `battencli/wire.go:451`).
  D5's second sentence is replaced with the reviewer's text, and a Fact records the evidence.
  The Notes bullet is deleted.
  The batten-site `lym batten cleanup --apply` way forward stays.
  The self-check then made the sentence conditional on a teardown that removes the pair (see below).
- **F15** (fixed): D7's reinterpretation now gives every profile that holds a tapestry block a jacquard block derived from that tapestry block.
  `defaults.jacquard` follows `defaults.tapestry`, or is dropped when that profile has no jacquard block.
  D7 now states that the reinterpretation runs after the follow (`walk.go`).
  It also states that a derived block differing from the base is kept on later reconciles (`follow.go:167-171`).
  The D7 criterion gains the tuned-profile and tuned-default cases.

## Self-check

The four lenses' findings were merged and judged together.
Every one was fixed.

- fix-claims MEDIUM, merged with fix-proposal MEDIUM and fix-way-forward LOW (finding 3): D5's no-leftover-branch sentence held only when the teardown removes the pair. A not-level pair ends Done and stays standing (`internal/battenshed/teardown.go:131-133`). Fixed: the sentence now opens "a teardown that removes the pair", and a new sentence names Worktree-Create's leftover-branch refusal and its way forward for a standing pair.
- fix-way-forward MEDIUM (finding 1): `lym loom start` was not shown to lift goto's pause. Fixed: the sequence now ends with `lym loom start`, then `lym loom resume`; the start commits the seed, and the resume lifts the pause, as the tapestry bullet's resume does.
- fix-way-forward LOW (finding 2): the Stuck reason did not spell out the re-seed. Fixed: the bullet gives the full commands and says the reason fills in the seed's path, driver and parent.
- fix-consistency LOW (finding 1): the re-seed way forward had no criterion that reaches it. Fixed: a tier 1 D4 criterion was added.
- fix-consistency LOW (finding 2): the D7 criterion's drop clause described the old rule. Fixed: it now drops `defaults.jacquard` when the profile `defaults.tapestry` names holds no jacquard block.
- fix-proposal LOW (finding 2): a re-seeded jacquard run keeps its first usage record and is counted under jacquard. Fixed: D9's bound names and accepts that case.

`lym loom validate-discussion` passes after every change.

## Disputed

None.

## Deferred

None.

## Rulings

None.
