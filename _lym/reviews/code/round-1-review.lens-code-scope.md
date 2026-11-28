```yaml
lens: code-scope
outcome: reported
```

Task: trace each change in the diff to the decisions in the record (code-scope lens).

**Decisions** (decision record alone: the goal points of `## Goal`; the record has no `Added after Discussion` entries)

- G1: give the darn writer prompt a rule for an entry whose deliverable is gate measurements. The rule either allows those calls with stated limits or has the writer ask its parent before running them (`_lym/discussion/decision-record.md:20-23`).
  - Built by C1 and C2. They take the goal's second option, asking the parent, and add a no-parent path. The goal allows either option.

**Changes**

- C1: the rule added to item 3 of `contracts/stencils/darn/darn-fragment-make-change-gated.md:41-45`.
  - Traces to G1 directly. The goal names this fragment as where the contradiction lives (`decision-record.md:9-10`).
- C2: the same rule added to item 3 of `contracts/stencils/darn/darn-fragment-make-change.md:23-27`.
  - Traces to G1. The goal asks for a rule in "the writer prompt", not only in the gated fragment. The ungated Step 4 is the same writer prompt under the other gate setting. Its per-commit gate rule (`darn-fragment-make-change.md:14`) sets no bound on a series of measurement calls, so the goal's "with what limits" applies there too.
- C3: both fragment header comments (`darn-fragment-make-change-gated.md:4-5`, `darn-fragment-make-change.md:2-3`).
  - Forced by C1 and C2: each header states the gate behaviour its fragment carries.
- C4: the `DarnFragmentMakeChange` and `DarnFragmentMakeChangeGated` doc comments in `contracts/stencils/stencils.go` (diff 002, lines 9 and 17-19), from both commits.
  - Forced by C1 and C2.
  - Commit c8e140d3b also rewords the existing reported-failure sentence into "asks its parent before diagnosing a task's reported failure, for the single run of the test it names". The meaning is unchanged; the sentence was re-broken inside a doc comment the traced change already edits. It is part of that forced edit, not new scope.
- C5: the `DarnTable` doc comment in `internal/darnengine/spec.go:93`.
  - Forced by C1 and C2: the doc states what the rendered Step 4 tells the writer in each gate mode.
- C6: the `measurementEntryRule` constant and its use in two `TestDarnSpec` rows (`internal/darnengine/spec_test.go:85-86`, `:121`, `:149`).
  - A test of C1 and C2, so forced by a traced change. Whether these asserts are justified under Test economy belongs to the code-tests lens and the handler, not this one.

**Findings**

None. Every change traces to G1 or is an edit G1's changes force, and G1 is built. No commit carries a `Pattern-Departure:` trailer, and none is needed.

**Tool calls:** none; the inherited context held the record, the diff pieces, both fragments and the commit log.
