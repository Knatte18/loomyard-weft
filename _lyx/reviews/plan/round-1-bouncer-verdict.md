---
verdict: APPROVED
rationale: "The round-1 report APPROVED with nine MEDIUM-or-lower findings, and all nine are fixed in the current plan files (verified per card); every Decision and Constraint in the decision record is carried, no card adds unlicensed scope, and verify runs unit and integration tests for all three targeted packages."
---

# Plan-Review round 1 — Bouncer verdict

The review's own verdict was `APPROVED` with one MEDIUM, six LOW and two NIT findings.
The fixer's report says all nine are fixed, so I judged the plan as it stands now, not as the reviewer saw it.

## Findings checked against the current plan

- **F1** (Card 3): the unarchived-leftover test now asserts "no local warp or `<slug>-weft` branch".
- **F2** (Card 3): the `SkipPush` test asserts that both of origin's branches are unchanged.
  The Intent says step 11's warp push is not gated by `SkipPush`/`SkipGit`, and that changing it is out of scope.
  I confirmed in `internal/fabricengine/add.go:231` that step 11 runs `push -u origin <warpBranch>` unconditionally, while `pushWeftBranch` (`weftwiring.go:140`) honours both options.
- **F3** (Card 3): the ImpactSummary now names `runAdd`, `battencli`'s create-worktree row and `hubforge.AddPair`, the two added `ls-remote` reads, and the earlier failure point.
  That is a real blast-radius conclusion.
- **F4** (Cards 3 and 5): the warp fast-forwardable test is now in Card 3 and uses `Remove(l, slug, false, true)`.
  `remove.go`'s doc comment confirms that `remote` deletes only the weft branch on origin and never adds a remote warp deletion, so Card 3 can reach the probe's pass branch without Card 5.
- **F5** (Card 5): the ImpactSummary covers the post-replacement rollback outcome and the `addBeforeWeftReplaceHook` var.
- **F6** (Card 2): a new integration case covers a valid tag with an empty lease SHA.
- **F7** (Card 3): the helper does the `cat-file -e` pre-check and then calls `gitrepo.Repo.IsAncestor`, which exists at `internal/gitrepo/ancestry.go:21` and is listed in `Uses`.
- **F8** (Card 2): the card now says to rewrite the header of `destroyremote_integration_test.go`.
  The header does claim that no test there asserts a remote deletion, so the update is needed.
- **F9** (Card 3): the card now names Card 5.

## My own pass over the rubric

- **Fidelity.** Every Decision maps to a card:
  - archive proof and `FETCH_HEAD`-only fetch → Card 3;
  - replace, never adopt, at step 12 with a lease → Cards 1, 2 and 5;
  - the four weft outcomes and the warp refusal → Card 3;
  - the refusal remedies and the bare `output.Err` → Cards 3 and 4;
  - skipping every probe under `SkipPush`/`SkipGit` → Card 3;
  - the `destroy.go` extension → Card 2;
  - no restore on rollback → Card 5.

  The Constraints are also carried: the chokepoint, the `CONSTRAINTS.md` sentence, the empty pre-flight record, `gitexec`, `newFabricFixture`, and integration tagging.
  Every acceptance criterion has a test, and the six `destroy.go` cases are split between hermetic and integration tiers as Test Tier Purity requires.
  The extra cases (primary weft refused, a `branchRequest` refusing the remote kind, an empty lease) follow from the record's own definitions.
  No card deletes or force-pushes a remote warp branch, and no card changes `Remove`.
- **Granularity.** The five cards are reviewable units.
  `probeWarpLeftover` and the ancestor helper are private to the file Card 3 creates, and the same card tests them.
- **`Custom`.** No card uses it.
- **Verify coverage.** `go test ./...`, plus `go vet` and `go test` with `-tags integration`, cover `gitrepo`, `fabricengine` and `fabriccli`, which are all the Go packages the plan targets.
  The `CONSTRAINTS.md` constraint tests run under `go test ./...`.
- **Docs.** `manifest/designs/` has no fabric module doc covering Add.
  `doc.go` and `CONSTRAINTS.md` match the record's docs scope.

## Focus departures

The report has no `## Focus departures` section.
Its focus verdicts answer all seven round-1 focus entries, and I ratify each one: two not-a-defect rulings and five findings (F1, F3, F4, F5, F6).
