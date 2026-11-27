# Fabric core: remove, destroy, cleanup, prune, remote teardown and pending teardown move to the core

`fabric-core-teardown` · Task · enhancement, fabric, testkit, shed, landing · draft

Decision record: [Sketch](decision-fabric-core-teardown.md)

Depends on: [`fabric-core-push-origin`](design-fabric-core-push-origin.md)

Filed from the Sketch `fabric-git-free-core`.

Takes: the push and origin part's core rules, installer and setters
Delivers: the rules of remove, cleanup, cleanupremotewarp, prune, archive, ownerarchive, remoteleftover, remoteteardown, pendingteardown and teardownlock in internal/fabriccore; the teardown child as a verb of its own that reads its own snapshot under the teardown lock, takes the handed facts and parent verdicts D2 lists, and pushes every archive tag before any origin delete as a core rule, with PushWithRetryReasons' loop kept inside one action; holdTeardown's in-process path reading the child's snapshot at the child's point; the child's entry point in internal/fabriccli taking the installed seams; destroy.go keeping its guards at execution with no rule moved into it; the pending-teardown record unchanged for fabric status, fabric prune and batten cancel; the D12 move of archiveparent's shed test; a test outside fabric that removes a hubforge pair unrefused with no seam of its own. Under the parent's added entry 'The restructure cuts fabric's test code, it does not only move it', this concern's tests, the heaviest of fabric's, shrink: a test is kept or moved only where it alone guards a rule, at the lowest tier that can check it, duplicates and the same rule tested at several tiers are deleted, not moved, and the change description lists every deleted test and, for each kept or moved test, the rule it alone guards
Exports: no new API: the teardown verbs keep their public signatures

The part changes these packages:

- internal/fabricengine (about 3900 changed lines)
- internal/fabriccore (about 3000 changed lines)
- internal/fabriclayout (about 150 changed lines)
- internal/testkit/fabrickit (about 150 changed lines)
- internal/fabriccli (about 200 changed lines)
- internal/shedengine (about 100 changed lines)
- internal/pairteardown (about 150 changed lines)
