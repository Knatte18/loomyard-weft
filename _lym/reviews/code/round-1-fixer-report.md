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
---

# Fixer report, round 1

## What changed

- F1: `Store.validateWrite` no longer refuses a dependency on a note or making a depended-on entry a note.
  `Dependencies.metBelow` judges a dependency on a note as not met whatever its status.
  The raddle `depends-note` check, its constant, its tests and its spec row are gone.
  `TestKindDependencyRules` has rows for a task and a note taking a note, a cycle through a note, demoting a depended-on task, and a done note staying pending.
  Consequence the review did not name: `MakeSketch` no longer refuses a depended-on entry, so I changed `sketch.go` comments, the `DependedOnEntry` subtest of `TestMakeSketch`, the `loomcli` integration test row for a depended-on entry, and deleted `TestSketchFinish_TellsTheParentOnlyOfARefusedSketch`, whose refusal can no longer be triggered.
  The commit carries a `Pattern-Departure:` trailer for it.
- F2: `Store.RemoveTask` strips the removed slug from every remaining entry's `depends_on` through a new unexported `removeEntry`.
  `MergeTasks` keeps dependencies on a slug it upserts again.
  `TestRemoveTaskStripsTheSlugFromDependsOn` covers the strip for a task and a note, and the merge keep.
- F3: `ly:board` skill line, `boardengine/doc.go` and `_lym/landing/summary.md` (file edited only, no git) now say a note may be depended on.
- F4: the duplicate `TestWriteSiblings` row is deleted.
- F5: the `BriefTask` doc names `Pending`.

A gate round then broke four comment spans that wrapped mid-sentence or ran to 120 characters, in `store.go`, `loomcli/doc.go`, `loomcli/wiring.go` and the `loomcli` integration test, at sentence boundaries; a second gate round rewrote the `loomcli/doc.go` sentence into two whole sentences; no behaviour changed.

Commits: F4 on its own, F1, F2, F3 and F5 as one group (shared `store.go`, `doc.go`), and one self-check follow-up.
Gate: `lym gate test` on `boardengine`, `raddleparser` and `loomcli` passed at tier0; the integration-tier test of `loomcli` was edited and vetted with `-tags integration` but not run.

## Disputed

None.

## Deferred

- Self-check fix-correctness 1 (MEDIUM): `RemoveTask` of an open entry a run-held task depends on now changes the held task's `depends_on`, which `checkRunLocks` reads as a scope change (`internal/boardengine/runlock.go:164-220`, forgiving only done removals).
  Whether such a removal is refused or forgiven is a decision for the parent; I changed nothing.
- Self-check fix-correctness 2 and fix-affected-set LOW: the Bolt replay's `replayRun.removeEntry` (`internal/boardengine/replay.go:757-771`) still refuses a removal while a dependent holds the slug, where the live board now strips.
  Aligning it needs a decision on how the replay treats the dependents' own replayed edits, so I left it.
- Self-check fix-proposal 1 (MEDIUM): a merge that folds `a` into a different slug `b` strips `a` from its dependents instead of repointing them to `b`.
  The review asked only to keep a dependency on a slug the merge upserts again; repointing is a product decision for the parent.
- The `sketchFinish` refusal path and `sketchRefusedNotice` in `internal/loomcli/wiring.go` remain, with the dependent-specific wording removed; no test reaches them now, and whether to delete them is for the parent.

## Rulings

None.

## Self-check

- fix-consistency, MEDIUM, stale `sketchRefusedNotice` wording: fixed.
- fix-consistency, LOW, `internal/loomcli/doc.go` unqualified refusal sentence: fixed.
- fix-consistency, LOW, test-file seed comment (unverified, no defect shown): no change, no finding.
- fix-consistency, LOW, "as prune does" against `Order`: fixed by rewording the `doc.go` sentence.
- fix-code-correctness, MEDIUM, run lock on a removal: deferred, see above.
- fix-code-correctness, LOW, replay twin: deferred, see above.
- fix-affected-set, LOW, replay twin: deferred, merged with the line above.
- fix-proposal, MEDIUM, merge into a different slug: deferred, see above.
- fix-proposal, LOW, layer of a dependent on a done note: disputed in the report only, `premise false`: `getDepth` gives the dependent depth 1 for a done note (`internal/boardengine/layer.go`, the `layerMap[slug] != ""` branch returns 0 for the note, so the dependent takes `maxDepth + 1`), so it reads as a letter layer, waiting, which agrees with `Pending`.
- fix-proposal, LOW, `doc.go` silent on prune of a done note: fixed.
