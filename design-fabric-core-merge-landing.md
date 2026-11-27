# Fabric core: merge-in, mid-merge, handover, takeover, integrate, task patch and discard move to the core

`fabric-core-merge-landing` · Task · enhancement, fabric, testkit, shed, batten, landing · draft

Decision record: [Sketch](decision-fabric-core-merge-landing.md)

Depends on: [`fabric-core-teardown`](design-fabric-core-teardown.md)

Filed from the Sketch `fabric-git-free-core`.

Takes: the teardown part's tree, with the installer and the shed adapter
Delivers: the rules of merge, mergeguards, mergelifecycle, mergestate, mergestateactive, mergestage, mergepaths, mergeerrors, midmerge, handover, takeover, takeoverwrite, integrateweft, taskpatch and taskbranchdiscard in internal/fabriccore; the shedengine.Status and shedengine.Takeover uses cut: OriginParentRunStatus and OriginBattenStatus return bytes and found, their callers decode through shed, ReadUpstreamTakeover and the result types that carried shedengine.Takeover carry fabric's own takeover entry of machine, subject and time filled by the seam's decoder, and the takeover write stays a state.WriteJSON patch of the one takeover key; the shed-module test written against today's code before the cut that pins the takeover write's status bytes, an undeclared key included, and the JSON-key parity of fabric's and shed's takeover types; the D12 moves of handover, takeoverrun and rejectedtakeover's shed tests to the shed module. Under the parent's added entry 'The restructure cuts fabric's test code, it does not only move it', this concern's tests shrink: a test is kept or moved only where it alone guards a rule, at the lowest tier that can check it, duplicates and the same rule tested at several tiers are deleted, not moved, and the change description lists every deleted test and, for each kept or moved test, the rule it alone guards
Exports: fabric's takeover entry type in internal/fabriclayout and the takeover decoder seam

The part changes these packages:

- internal/fabricengine (about 3000 changed lines)
- internal/fabriccore (about 2200 changed lines)
- internal/fabriclayout (about 200 changed lines)
- internal/testkit/fabrickit (about 100 changed lines)
- internal/shedengine (about 900 changed lines)
- internal/battencli (about 100 changed lines)
- internal/landingshed (about 100 changed lines)
