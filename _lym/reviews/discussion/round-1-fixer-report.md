---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed-differently
    reason: "Both cases go through the one-time reinterpretation; the alternative D6 way forward `lym config profiles --set <profile>.jacquard=<block>` was not taken, because that flag syntax was not found in the code."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed-differently
    reason: "The check stats the status file before loomshed.Seed and judges only itself, so a refused first start writes no status file; the proposed wording 'absent when bootstrap reads it' named a read bootstrap does not make (sharedbootstrap.go seeds through Seed's lock alone)."
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed-differently
    reason: "The proposed sentence listed only the batcher and drift checks as build-only; Preflight also runs stencils and hardener checks, so the rationale now says every check other than the worktree and fabric checks guards build-row config."
  - id: F12
    disposition: fixed
---

# Round 1 fixer-report

Every change is in `_lym/discussion/decision-record.md`.
F8 and F9 are `decision`-class findings, held by the parent and left out here.

## Changes

- F1: D5's two sites now carry their own ways forward.
  Batten's guard keeps the board upsert.
  `lym loom start` names D4's jacquard sequence for a pair batten made.
  The clause saying a start with no status file can take the upsert is dropped.
  The D5 criterion now reads "each site's message carries its own way forward, and the test reaches both".
- F2: D4 now has the RecordWrite-path refusals in `internal/loomcli/decision.go` and `recordWritePending` name the recipe's record or seed row through `loomrecipe.EngineRow`.
  A tier 1 criterion is added: on a jacquard run with an uncommitted record, the refusal names `lym loom goto --to Record-Seed`.
  The `PATTERN-refusal-way-forward` constraint now names these refusals too.
- F3: D5 now has `startClearsMotifRun` admit a start whose recipe needs a stored record, so the start is judged as fresh.
  A tier 1 criterion is added: after a done motif run, a jacquard start of the same entry starts.
- F4 (fixed differently, see frontmatter): D7 adds a one-time reinterpretation, registered in `internal/configreg` beside the arras one.
  It drops a `defaults.jacquard` that names a profile with no jacquard block.
  It gives an operator-added profile that holds a tapestry block a jacquard block.
  The D7 criterion covers both cases.
  The unverified fact holds: reinterpretations are registered in `configreg.go:171`.
- F5: D2's equal-keys sentence and the D1/D2 criterion now except `entry`.
- F6: D3 admits an empty `on_done` on the last block row only with `block_gate: always`, and refuses any other gate with a way forward.
  The Bound now states that no skip can reach the empty exit.
  The D3 criterion now includes the refusal.
- F7 (fixed differently, see frontmatter): the `lym loom start` check runs on a first start only.
  It stats the status file before `loomshed.Seed`, and a re-run over an existing status file passes over it.
  The D5 criterion now says such a re-run is not refused.
- F10: the D11 Review criterion now reads "no file of the diff writes Bolt's `heddle.json` or a nickname line."
- F11 (fixed differently, see frontmatter): D2's rationale now separates the worktree and fabric checks from the build-only checks.
- F12: D9's Bound now accepts the case of an entry flagged discussed whose record file is missing.

## Self-check

- fix-consistency, LOW, the `PATTERN-config-layers` constraint was stale after D7's reinterpretation: fixed as proposed.
- fix-consistency, LOW, D5's Bound "reads the board once and nothing else" left out the done-run read: fixed as proposed.
- fix-claims, MEDIUM, bootstrap never reads the status file before Seed: fixed with own change.
  The check now stats the status file only to judge itself, and seeding stays under Seed's lock.
  The lens's options of judging inside the lock or after Seed were not taken, because both would leave a status file behind a refused start.
- fix-claims, LOW, D2's rationale left out the stencils and hardener checks: fixed as proposed.
- fix-proposal, MEDIUM, the reused motif clear leaves the motif task branch: fixed with own change.
  D5 states that a jacquard start meets the branch as a raddle start does.
  The open fact goes to the plan writer's notes, with the `ClearDoneRun` path as the fallback.
  The proposed switch to `ClearDoneRun` was not made, because a raddle start goes through the same clear and the create-row handling was not read.
- fix-proposal LOW and fix-way-forward LOW (merged), `lym batten cancel` cannot act on a pair batten did not make: fixed with own change.
  The `lym loom start` way forward now names two cases.
  A batten-made pair takes D4's sequence.
  Any other pair re-seeds with `lym shed seed self --recipe tapestry`, the form `disagreeingSeedRefusal` names.
- fix-way-forward, MEDIUM, after the upsert a done motif run still refuses a tapestry start: fixed as proposed.
  The batten-site way forward names `lym batten cleanup --apply` first when the slug holds a done motif run.

## Disputed

None.

## Deferred

None.

## Rulings

None reached this round.
