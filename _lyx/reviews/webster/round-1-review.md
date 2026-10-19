---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/fabricengine/destroy_test.go:501
    summary: A test comment names removePortal, deleted by card 4, as an idempotent teardown site; a sibling comment still says portal/launcher teardown
    fix: In internal/fabricengine/destroy_test.go:501 drop removePortal from the list "(removePortal, removeLaunchers, Remove's already-absent weft worktree)"; in internal/fabricengine/destroy_containment_toctou_integration_test.go:10 change "portal/launcher teardown" to "launcher teardown".
    origin: lens:code-affected-set
  - id: F2
    severity: LOW
    class: scope
    location: PATTERN.md:31
    summary: The diff rewords three PATTERN entries the declared changes do not list (push-both-sides, recipe-setting-lookup, batcher-registry)
    fix: In PATTERN.md restore the wording main carries for PATTERN-push-both-sides, PATTERN-recipe-setting-lookup and PATTERN-batcher-registry, leaving only the two declared containment entries changed; or declare the rewording if it is intended.
    origin: lens:code-pattern-fit
---

### [LOW] Stale removePortal reference in test comments

**Location:** internal/fabricengine/destroy_test.go:501, internal/fabricengine/destroy_containment_toctou_integration_test.go:10
**Issue:** Card 4 deletes `removePortal`, but a comment still lists it as a teardown site, so a rename search no longer finds it.
A sibling comment still describes "portal/launcher teardown".

### [LOW] Undeclared PATTERN.md rewording

**Location:** PATTERN.md:31
**Issue:** Commit `acf5697d3` rewords `PATTERN-push-both-sides`, `PATTERN-recipe-setting-lookup` and `PATTERN-batcher-registry`.
The declared PATTERN changes name only `PATTERN-fabric-write-containment` and `PATTERN-hub-containment`, and `main` still carries the old wording of the three.

## Rejected

- Code-tests: `TestWriteHubWorkspace` re-pins unchanged-bytes behaviour: card 1 lists it under Edit, so the plan retains it.
- Code-tests: `TestHubWorkspacePathAndFolders` asserts only a path join: card 1 lists it under Edit, and it carries a deliberate `//testtiming:keep` marker.
- Code-tests: helper-level `writeHubWorkspace` tests: card 1 lists both under Create, and `os.Root` refusal cannot be reached through the public function.
- Code-tests: no test for `lyx ide spawn <prime>` returning the regeneration error: `checkPrimeWriteFailure` in `spawn_hub_integration_test.go:267` pins it.
- Code-tests: failed local-mark read in `restoreLaunchers` unpinned: the mark read cannot be injected without a fake, and the lens itself called it hard to inject.

## Holistic notes

All focus items hold:
- Every verb call site logs a Warn naming `lyx fabric reconcile` through `regenerateAfterVerb` and never fails or rolls back; only `ideengine.Spawn` returns the error.
- The hub workspace file and lock scratch dir are written through an `os.Root` at the hub.
- Both declared PATTERN changes land with their declared text.
- No retired portal symbol remains in non-test code, `doc.go` or docs, apart from F1's comments and a measurement report.
- Lock-wait tests use `RegenerateHubWorkspaceWithinForTest`.
