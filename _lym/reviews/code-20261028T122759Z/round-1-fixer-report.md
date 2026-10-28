# Round 1 fixer-report

## Changes

- F1 (LOW): added five rows to `TestRefScanner_Matches` pinning that gate blanking applies only in command position and ends at `;`, `|`, `||` and a newline. Commit 2117638a.
- F3 (LOW): `gateSegmentPattern` now keeps a redirect ampersand (`2>&1`, `>&2`, `&>`) inside the segment, with the proposed alternation, and a row `lym gate test 2>&1 --card 6-prime-weft` (want false). The doc comment says so. Commit 2117638a.
- F2 (LOW): added `TestRecoverSpawnOrAttach` rows for the dirty-tree refusal, a HEAD that does not descend from the start, and a `HeadSHA` read failure that is returned and is not `ErrRecoveryNeedsFresh`. Commit ffc69d5f.
- F5 (NIT): added a row for a committed batch with a nil `Deps.ReadOnly`, expecting the batch-start route. Commit ffc69d5f.
- F4 (LOW): narrowed the card-id wording to "other than a `lym gate` invocation" in `internal/planparser/fabricref.go` (doc comment and finding text), `internal/planparser/doc.go` and `contracts/specs/loom-plan-spec.md`, and added a planparser row for a Verify gate command naming a suffix-ending card ID (passes). Commit f2d46077.
- F4 also: corrected the plan-check sentence in `_lym/landing/summary.md` to say only a Verify span with a gate command is no longer flagged. That file is under `_lym`, so it is edited in the working tree and not committed by this round.

`lym gate test` over `./internal/fabricengine ./internal/planparser ./internal/websterengine` passed (tier0, all targets).

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings. It noted, without raising it, that the `IsAncestor` and `UncommittedPaths` read errors have no row; the review asked for a single read-error row.

## Disputed

None.

## Deferred

None.
