```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the board entry's `## Goal` points.

**Decisions**
The answer key is a decision record alone, and it holds no `Added after Discussion` entries.
- **G1:** give the measurement rule a path for a run with no parent, so the measurements are still produced or the run stops with a way forward.
- **G2:** add a sentence to the gated fragment that bars subagents from measurement calls.

**Per change**
- **C1:** both Step 4 fragments now name the no-parent stop report's way forward as `lym loom decision add` with an allowing entry, then `lym loom resume`. On a task a Code-Review round has run on, it is a board note for a new task, then `lym loom resume`. Traces to G1.
  - `contracts/stencils/darn/darn-fragment-make-change.md:38-40`
  - `contracts/stencils/darn/darn-fragment-make-change-gated.md:54-56`
- **C2:** the gated fragment adds "Run them in this session: no subagent runs them." Traces to G2.
  - `contracts/stencils/darn/darn-fragment-make-change-gated.md:46`
- **C3:** `internal/darnengine/spec_test.go` gains the `measureNoSubagent` and `measureNoParentWayForward` constants. Both `TestDarnSpec` rows assert them in the rendered prompt.
  - This is forced by C1 and C2: the entry is labelled `bug`, so it needs a test that fails before the fix.
  - Whether that test passes the rubric's test-economy rules belongs to the code-tests lens.

**Per decision**
- **G1:** built by C1, with C3.
- **G2:** built by C2, with C3.

**Findings**
None. Every change traces to a goal point or is forced by one, every goal point has a change that builds it, and the diff makes no change beyond the goal.

One point outside my lens, for the correctness lens or the handler: in the closed-scope branch, nothing tells the resumed writer that the board note settles the stop. The note fragment's rule still reads "With no such entry, run none of them and stop." Step 3 of `contracts/stencils/darn/darn-template-write.md:53-55` has such a settling sentence, but it covers only Step 3's outdated-entry stop. So the run may stop again after the resume.

Tool calls: none.
