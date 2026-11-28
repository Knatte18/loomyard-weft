# Fabric core: push classification, retries, origin reconcile and failed-push records move to the core

`fabric-core-push-origin` · Task · enhancement, fabric, testkit, shed, board, pattern · running · Plan-Write

Decision record: [Sketch](decision-fabric-core-push-origin.md)

Filed from the Sketch `fabric-git-free-core`.

Takes: the foundation part's layout, core types, reader and executor loop
Delivers: the rules of pushclassified, pushanchored, coalesce, failedpush, origin, originreads, originreconcile, originreconcileended, originreconcilemoves, pendingflags, pusherreads, pushlockholder and remoteonly in internal/fabriccore as phase and action-list rules over made-up snapshots, with the retry loop as wait actions and OriginSeams' teardown as a seam-call action; the engine running them, with rejectionRetryKind's error-to-class mapping and the live-pair probe before an origin move kept in the engine; these files' shedrun and shedengine imports cut: run paths, the prime run lock and the status codec reach fabric through required seams, the status readers return bytes their callers decode; D13's seam installer in internal/fabriccli, its call from cmd/lym/main.go and internal/hubforge's one-time call, each required seam's setter with its restore, the unset-seam refusal row and engine test, and shed's adapter with its test in the shed module; the D12 moves of this concern's boardkit and boardengine tests, such as pushclasses, pushanchored and originreconcile*, to internal/boardengine; PATTERN-fabric-boundary's background recorder clause restated as D4 words it, since this part's executor first records entries. Under the parent's added entry 'The restructure cuts fabric's test code, it does not only move it', this concern's tests shrink: a test is kept or moved only where it alone guards a rule, at the lowest tier that can check it, duplicates and the same rule tested at several tiers are deleted, not moved, and the change description lists every deleted test and, for each kept or moved test, the rule it alone guards
Exports: the required-seam installer and setters later parts add seams to; the push and origin core rules ReconcileOrigin callers reach through the unchanged public verbs

The part changes these packages:

- internal/fabricengine (about 4100 changed lines)
- internal/fabriccore (about 2500 changed lines)
- internal/fabriclayout (about 200 changed lines)
- internal/testkit/fabrickit (about 150 changed lines)
- internal/fabriccli (about 250 changed lines)
- internal/hubforge (about 20 changed lines)
- cmd/lym (about 30 changed lines)
- internal/shedengine (about 400 changed lines)
- internal/boardengine (about 1200 changed lines)
- pattern (about 20 changed lines)
