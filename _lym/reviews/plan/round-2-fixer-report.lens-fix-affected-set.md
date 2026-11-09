```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set. For each finding whose premise is a set, I reran its search and checked that the fix covers every member. I found nothing.

Findings left unchecked:
- **F4:** I did not search the full `TestAdopt` table for other rows that repeat the `TestRunWorker` rows. The rows left in `TestRunWorker` all test the resume, the lock or the request merge, which `TestAdopt` cannot reach.
- **F5:** I did not rerun its search. Its premise is three named pairs of tests, not a set to sweep.
- **F8:** I did not rerun its search. The fix states one general rule for the existing `reentry` text, so it covers every row that asserts that text.

Checked:
- **F6:** All of the stale adoptcli comments outside tests are covered:
  - the `adopt.go` header and the `resumeSlug`, `runBeamSteps` and `heldBeamWayForward` docs, which the fix now names;
  - the `deps.go` header, which the fix now names;
  - the `holds.go` header and the hold docs, which card 6 already rewrites or deletes.

  The `Green` seam comment at `check.go:119` is in card 4's scope.
- **F2:** Every return of the outcome gathered so far in `adopt` (`adopt.go:254-396`) is the error of some step. The fix stores `StoppedPart` for any step that fails, so all of these returns are covered: the reconcile error, the unreachable origin, the shared stop, the moved-runs read, `projectOf`, `wouldStop`, the pause, the deploy, `SetOwners` and the read-back. The standing-move refusal returns an empty outcome today as well.
- **F7:** Three comments describe the scratch directory: `fabricengine/junctionnames.go:188`, `fabricengine/doc.go:1122` and `lymdirs/dirs.go:40`. Card 3 now covers all three. The mentions in adoptcli's `doc.go` belong to card 10's rewrite.
- **F1 and F3:** The fix names each member of these sets: the inputs `reconcileWarnings` needs, and the places that read `Subjects` (the seeding in `runSteps` and the slug list in `envelopeOf`).

Tool calls:
1. Grepped the comments of the adoptcli files outside tests for the old green-end and hold wording, to rerun F6's search; my context held only the `adopt.go` comments and the `deps.go` header.
2. Grepped the comments that describe the adopt scratch directory and the `doc.go` files that mention it, to rerun F7's search; my context held only `lymdirs/dirs.go:40`.
