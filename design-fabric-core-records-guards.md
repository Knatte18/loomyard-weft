# Fabric core: anchors, records store, commit, trailers and index move to the core; import guards, PATTERN close-out and remeasure

`fabric-core-records-guards` · Task · enhancement, fabric, testkit, pattern, docs · draft

Decision record: [Sketch](decision-fabric-core-records-guards.md)

Depends on: [`fabric-core-add-clone`](design-fabric-core-add-clone.md)

Filed from the Sketch `fabric-git-free-core`.

Takes: the add and clone part's tree, every other concern moved
Delivers: the rules of anchor, anchorchange, anchormarker, anchormove, anchorstop, anchortarget, commit, committedfile, commitweftpaths, index, moverecord, recordsgit, recordslayout, recordsprobe, recordsreads, snapshotuncommitted, trailer and trailercommit in internal/fabriccore, Commit's secret-path refusal as a pre-lock snapshot fact the engine fills from internal/secretpath's matcher; the cross-verb livestate, onerepository_lifecycle and droplocal tests and export_test.go reduced to what no other test guards; the tier0 guard on test imports of the three fabric packages (D12) and the tier0 guard on their production dependency closure (D13); PATTERN-fabric-boundary's rule line and the new PATTERN-fabric-core entry beside the guards they cite; the remeasure doc under docs/research/speed/, measured in one session against a checkout of the commit before the foundation part that the implementer asks its parent for, which also records fabric's test lines before and after. Under the parent's added entry 'The restructure cuts fabric's test code, it does not only move it', this concern's tests and the cross-verb tests shrink: a test is kept or moved only where it alone guards a rule, at the lowest tier that can check it, duplicates and the same rule tested at several tiers are deleted, not moved, and the change description lists every deleted test and, for each kept or moved test, the rule it alone guards
Exports: the two import guards

The part changes these packages:

- internal/fabricengine (about 2200 changed lines)
- internal/fabriccore (about 1800 changed lines)
- internal/fabriclayout (about 150 changed lines)
- internal/testkit/fabrickit (about 100 changed lines)
- pattern (about 100 changed lines)
- docs/research/speed (about 250 changed lines)
