---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/fabricengine/shortnamebinding_clone_integration_test.go:94
    summary: The new ResetWithDisagreeingShortnameKeepsHub subtests run serially against the shared existing hub with no comment naming that shared fixture, which PATTERN-test-speed requires when a test omits t.Parallel
    fix: In TestCloneHub_ShortnameRecordLifecycle, add one comment above the t.Run subtests (or at the top of the function) stating that the subtests do not call t.Parallel because they share one existing hub fixture (firstParent and first) and run in order, since ResetKeepsRecord tears that hub down and rebuilds it. Add no t.Parallel call.
    origin: handler
  - id: F2
    severity: MEDIUM
    class: decision
    location: internal/fabricengine/clone.go:240
    summary: A --subpath that disagrees with the recorded anchor, or names a directory absent from the cloned warp, is still refused only after --reset has removed the existing hub; the change description records this as a design decision, not a parent-recorded narrowing of the entry's every-argument wording
    fix: Leave the code in place. Defer this finding to the run's parent, whose only sound step is recording the narrowing to shortname, warp binding and old-order guard (or refusing it and requiring the subpath check before the reset) with lym loom decision add; it lies outside the fixer's write surface.
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: scope
    location: internal/fabricengine/warpbinding_clone_integration_test.go:238
    summary: No test runs --reset with a warp binding mismatch or the old-order bootstrap guard against an existing hub, so moving the two-argument removal back before the probe would go unnoticed for both refusals; only the shortname refusal is pinned
    fix: Add rows that run CloneHub with Reset true against an existing hub, in the two-argument form, for a warp URL that disagrees with the recorded binding (assert errors.Is ErrWarpBindingMismatch) and for the old-order guard (assert the refusing to bootstrap text), each asserting the hub survives. Put them as rows in one table beside ResetWithDisagreeingShortnameKeepsHub in shortnamebinding_clone_integration_test.go, or as Reset variants of TestCloneHub_ConflictLeavesNoHub and TestCloneHub_OldOrderInvocationIsRefused, reusing the existing fixture helpers and sharing one hub setup.
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: consistency
    location: internal/fabricengine/livestate_refusal_selftest_test.go:310
    summary: Test comments still name resetHub's own pre-flight as the offline refuser of a non-hub target in the two-argument form, which checkResetTarget now raises from CloneHub
    fix: Reword the comments to name checkResetTarget, or the offline pre-flight, as the refuser, without line numbers. Edit livestate_refusal_selftest_test.go around lines 310 and 321, livestate_verbs_test.go around lines 51 and 992, and livestate_matrix_test.go around line 313.
    origin: lens:code-affected-set
  - id: F5
    severity: NIT
    class: consistency
    location: internal/fabricengine/clone.go:236
    summary: The shared-block comment says the hub-exists check repeats for the two-argument form, but with Reset that form ran the non-hub check above, not a hub-exists check; the asymmetry sentence beneath it reads as orphaned
    fix: Reword the comment at the top of the shared block to say the offline check of the two-argument form repeats here, as the non-hub refusal with Reset and as the hub-exists check without it, and keep the asymmetry sentence attached to the one-argument remark that follows it.
    origin: lens:code-correctness
---

### [BLOCKING] Serial subtests carry no comment naming the shared fixture

**Location:** internal/fabricengine/shortnamebinding_clone_integration_test.go:94
**Issue:** `PATTERN-test-speed` ("Tests run in parallel") requires a comment, at the test or once atop its file, naming the shared state when a test does not call `t.Parallel`.
The new `ResetWithDisagreeingShortnameKeepsHub` subtest and its two row subtests call no `t.Parallel`, because they read the one existing hub the parent test built and run before `ResetKeepsRecord` replaces it.
Nothing in the function or the file header says so.
The sibling subtests share the omission, but a written rule outranks a convention inferred from the surrounding code.

### [MEDIUM] A refused --subpath under --reset still removes the existing hub

**Location:** internal/fabricengine/clone.go:240
**Issue:** The reset now runs after the probe, but the `--subpath` checks (`adoptRecordedAnchor`, and the create-path directory check) run at step 8, after the clone.
A disagreeing or nonexistent subpath is therefore refused only after the old hub is gone, and `teardownHub` removes only the fresh hub.
The amended entry's Expected line says `--reset` validates every argument before removing anything, with shortname and binding as the named examples.
`_lym/landing/summary.md` states the gap as the writer's own design decision, not as a correction the parent recorded, so it is filed for the parent to settle.

### [MEDIUM] Binding mismatch and old-order guard under --reset are untested

**Location:** internal/fabricengine/warpbinding_clone_integration_test.go:238
**Issue:** The behavior the diff changes, that every probe-time refusal leaves the existing hub in place, is pinned only for the shortname refusal.
`TestCloneHub_ConflictLeavesNoHub` and `TestCloneHub_OldOrderInvocationIsRefused` run without `Reset` and without an existing hub, so they cannot fail if the two-argument removal moves back before the probe for these two refusals.
The entry names the binding explicitly, and the summary and the `CloneHub` doc comment claim the old-order guard precedes the removal.

### [LOW] Test comments name the wrong refuser of a non-hub target

**Location:** internal/fabricengine/livestate_refusal_selftest_test.go:310
**Issue:** In the two-argument form the non-hub refusal is now raised by `checkResetTarget`, called from `CloneHub`, and `resetHub` never runs for that target.
The comments listed in the fix still say `resetHub`'s own pre-flight refuses it, and some pin line numbers that were already stale.

### [NIT] Shared-block comment describes the wrong check

**Location:** internal/fabricengine/clone.go:236
**Issue:** The comment says the hub-exists check repeats for the two-argument form, which ran it offline above.
With `Reset` that form ran `checkResetTarget`, so the sentence holds only for the non-reset path, and the following asymmetry sentence no longer sits beside the remark it explains.

## Rejected

- Warp and weft clones still run after the reset, so an unreachable warp URL still costs the hub (lens code-correctness, LOW): the entry names the shortname and the binding checked against the weft remote, and the failure ordering is unchanged by the diff.
- The `livestate_doc_test.go` mutation record names `resetHub`'s pre-flight (lens code-affected-set, LOW): the record is a measurement report of one past run and goes stale by design.
- One-argument form without `--reset` has no test for the hub-exists refusal (lens code-tests, LOW): the refusal behaves as before the diff and the gap is pre-existing, so it is not a behavior the diff adds.
- The one-argument `OneArgument` row passes without the diff (handler): it pins the entry's contract for the second form and duplicates no existing test.
