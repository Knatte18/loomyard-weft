```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. Trace each change in the diff to the board entry's goal points, and each goal point back to the changes that build it.

**Decisions (the `## Goal` board entry; the record has no `Added after Discussion` entries)**
- **G1.** For an entry labelled `bug`, Discussion-Write's stencil asks for and records the observed fault and how to reproduce it.
- **G2.** It asks for and records the root cause, or what is still unknown about it.
- **G3.** It asks for and records the test that will fail on the merge base and pass after the fix, or why no test seam exists.
- **G4.** The decision record then states up front what the red-first gate later checks.

**Changes → decision**
- **C1.** Stencil bug paragraph (`contracts/stencils/loom/loom-template-discussion.md:136-140`): has the writer settle the fault and reproduction, the cause or what is unknown, and the fail-on-base test or why no seam exists, and points at the spec's rule. Traces to G1, G2 and G3, the "asks for" half.
- **C2.** Spec section `### A bug entry's record`, first block (`contracts/specs/loom-discussion-spec.md:335-343`): places the fault and reproduction in `## Goal`, the cause in `## Constraints`, and the test or no-seam reason in `## Acceptance criteria`. Traces to G1, G2 and G3 ("records"), and to G4 by its opening line. Putting the format in the spec rather than the stencil is forced by PATTERN-stencil-ownership "Producer pointers": the stencil never restates a format contract.
- **C3.** Spec waiver sentence (`contracts/specs/loom-discussion-spec.md:345-346`): a record's statement that no seam exists does not lift the gate; only the parent's `No test seam` added entry does. Traces to G3, as the bound on its "why no test seam exists" branch. G3 lets a record name a reason the gate otherwise needs a parent decision for, and this sentence keeps that reason from reading as a waiver. That matches `internal/loomshed/redfirstgate.go:58`, which lifts only on an added entry's title.
- **C4.** Rubric bullet (`contracts/stencils/loom/loom-rubric-discussion-review.md:451-455`): flags a bug record that omits the fault, the cause or the test. Traces to G4: a review check that the record does state them up front. The entry does not name the rubric, but the bullet only enforces what G1–G4 require the record to hold. It adds no requirement beyond those three, so it is not unrequested scope.

**Decision → changes**
- **G1:** C1, C2, C4
- **G2:** C1, C2, C4
- **G3:** C1, C2, C3, C4
- **G4:** C2, C4

**Findings:** none. Every change traces to a goal point, and every goal point is built.

Checked and not filed: G1 says the stencil "asks for" the fault and cause, but C1 says these are researched facts and never interview questions (`loom-template-discussion.md:139`). That still builds G1: the stencil requires the writer to establish and record them. It also agrees with Step 2's standing rule (`loom-template-discussion.md:113-118`) that a factual question is researched and never put to the interview.

Tool calls: none.
