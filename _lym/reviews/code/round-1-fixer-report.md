# Fixer-report, round 1

## Changes

- F1 (MEDIUM), fixed as proposed: `childProjects` in `internal/boardengine/render.go` counts an open project under a done project as under the root, so it gets a README section. The fourth golden row gains an open project `orphan` under the done project `old`. Commit e9032bde.
- F2 (LOW), fixed as proposed by rewording: the `doc.go` README paragraph and the `childProjects` comment say root-level project sections run by handle, nested ones in their parent's order key, then by handle. Commit e9032bde.
- F3 (LOW), fixed as proposed: in the Ship project of the fourth golden row, `first` has low priority and `second` has high priority and a body. The expected tables show the `L` and `H` cells and the linked slug. Commit e9032bde.
- Follow-ups from the self-check, commits b336a758 and the docs commit after it: `doc.go` and `docs/overview.md` claim the entries of every open project, not every entry, and the `render_test.go` doc comment states the root-level order and the done-parent case.

## Self-check

- fix-consistency, LOW, doc claim "every open entry shows in the README" overreaches for tasks and notes under a done project: fixed with my own change. The proposal's alternative, a section for such entries, widens the render beyond the review's premise; the claim is reworded instead.
- fix-proposal, LOW, same overclaim: merged with the finding above.
- fix-proposal, LOW, no golden for `project-old.md` and `project-orphan.md`: disputed, premise false; see Disputed.
- fix-affected-set, LOW, `render_test.go` comment imprecise: fixed as proposed. Its `docs/overview.md` twin: fixed with the same wording change as the doc.go claim.
- fix-code-correctness: no findings.

## Disputed

- Self-check, fix-proposal LOW, `project-old.md` Projects section not pinned: premise false. The fix changes `childProjects` only for the root call; a done project's file calls it with a non-root project, so its Projects section is unchanged by this round and was not added by it. Evidence: the `under` clause in `childProjects` is false when `project.Slug != ""`.

## Deferred

None.
