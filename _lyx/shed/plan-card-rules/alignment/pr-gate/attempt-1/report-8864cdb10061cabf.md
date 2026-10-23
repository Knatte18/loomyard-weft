---
verdict: APPROVED
---

## Alignment at PR-Gate: plan-card-rules

The run was judged against its board entry's four scope points and decisions D1 to D9 and the added parent decision of 2026-10-23, using the diff of pull request 850 at `5cb1f3cf` against `main` (merge base `653665a5`, main's head).

- Headroom check (D1, D2): `internal/pattern/projection.go`, `internal/loomshed/patternplan.go`, and the merge-base reader `patternlint.OverviewAt` wired through `planContextOf` and `validate-plan`.
- `Extends` label and pair rule (D3 to D6): `internal/planparser/extends.go` and `pairrule.go`, the readers in `planglyph`, `batcher` and `websterengine/sequence.go`, `tools/plandag` with the `E` mark and `-additive-guess`, the plan-review rubric and the plan-writer stencil.
- Amendment owner (D7, D8): `lyx loom decision add --amends-plan` and the amendment record at `_lyx/loom/amendment.yaml`, the settle record's round start, the settled-segment guard, the skip seam, removal by Plan-Bouncer and PR-Rework, the answer-key instruction, the recipe's `amendment_target` key, the arbiter and orch stencils, and Webster's plan-conflict way forward.
- Failure trace (D9): the new `internal/failuretrace` package, `planglyph.EncloseAt`, the record fields in `verifytree`, the landing seam, and the note rendering.
- The parent's 2026-10-23 decision to port main's reed pane-tail fix is met: the branch's `fix:` commit leaves no `internal/reedengine` change against `main` once main is merged in.

The constraints hold.
`PATTERN.md` is unchanged, and Enclose is added to the background file `pattern/PATTERN-glyph-conversion-chokepoint.md` only.
`go.mod` is unchanged, since quarry v0.4.0 is already pinned.
The peripheral changes all serve the decisions: the tree-sitter link allowlist entry for `internal/failuretrace`, the `envkit` nil-legal and rework fixture fields, the sandbox suite's label list, and the `docs/overview.md` module line.
The Plan-Review carry-over `card19-arbiter-doc-go-stale` is addressed: `internal/loomcli/doc.go` now describes `amendment_target`.
The open risks (traces that name no card, the hub orch rule outside this repo, a profile without the Plan-Review perch) are recorded as out of scope or follow-ups and need no change here.

No board entry is made obsolete.
`parallel-batch-parts` and `weaver` stay open: the decision record puts them out of scope, and this run only places the pair rule so they can call it.

## Overlaps

- Peer `batten-cancel-verb` (row 16, Merge-In): both runs change `contracts/stencils/orch/orch-template-role.md` and `docs/overview.md`, at different places.
  This run adds the `--amends-plan` route to the orch stencil's escalation, fixing and decision bullets, and in the overview it adds the `failuretrace` module line, the `EncloseAt` mention under planglyph and the `--amends-plan` flag.
  The peer adds `lyx batten cancel` to the stencil's run-handling list and to the overview's batten line.
  The peer also lists `internal/reedengine` files from the same main fix a1200c20 that this run ported; this run's diff against `main` holds no reedengine change, so the two do not conflict.
  This is a text-only overlap with no design conflict.
  Landing order: this run is at row 21 and the peer is at row 16, so this run lands first and `batten-cancel-verb` takes its changes at Merge-In.
- Parent commits: `lyx loom peers` lists none.
