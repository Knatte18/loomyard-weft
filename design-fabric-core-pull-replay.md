# Fabric core: pull, replay, bolt, reconcile, parent tips and warp forward move to the core; stencil reads leave fabric

`fabric-core-pull-replay` · Task · enhancement, fabric, testkit, shed, stencil, board, heddle · draft

Decision record: [Sketch](decision-fabric-core-pull-replay.md)

Depends on: [`fabric-core-merge-landing`](design-fabric-core-merge-landing.md)

Filed from the Sketch `fabric-git-free-core`.

Takes: the merge and landing part's tree, with the takeover entry type and the shed adapter
Delivers: the rules of pull, hubpull, replay, bolt, boltreplay, reconcile, parenttips, codetipsettle, upstreamrun, stencilcommit and warpforward's fabric side in internal/fabriccore; the code-tip settle handing the seam the previous transition's status bytes with the status and lock paths, shed's adapter writing them through state.WriteJSON, and the shed-module test before the cut that pins its status bytes; upstreamrun's done verdict asked of the seam's decoder; replay's and pull's shedrun uses through the run-path seam; StencilSource and StencilBaseByStamp moved to internal/stencilsource (new) over fabricengine.HeadContains and the path-taking reads fabric adds (commit exists, ancestor of, a path's revisions, a file at a revision), with internal/stencilcli and cmd/lym's stencilseed retargeted and their two integration tests moved with them; the D12 moves of the bolt, pull and replay tests that drive boardengine, boardkit or heddleengine. Under the parent's added entry 'The restructure cuts fabric's test code, it does not only move it', this concern's tests shrink: a test is kept or moved only where it alone guards a rule, at the lowest tier that can check it, duplicates and the same rule tested at several tiers are deleted, not moved, and the change description lists every deleted test and, for each kept or moved test, the rule it alone guards
Exports: internal/stencilsource's StencilSource and StencilBaseByStamp; fabricengine's commit-exists, ancestor, path-revisions and file-at-revision reads

The part changes these packages:

- internal/fabricengine (about 3200 changed lines)
- internal/fabriccore (about 2200 changed lines)
- internal/fabriclayout (about 150 changed lines)
- internal/testkit/fabrickit (about 100 changed lines)
- internal/stencilsource (about 700 changed lines)
- internal/stencilcli (about 40 changed lines)
- cmd/lym (about 30 changed lines)
- internal/shedengine (about 400 changed lines)
- internal/boardengine (about 900 changed lines)
- internal/heddleengine (about 200 changed lines)
