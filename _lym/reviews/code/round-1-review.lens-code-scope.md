```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the board entry's goal points.

**Decisions**, from the board entry under `## Goal`. The record has no `Added after Discussion` entries.
- **G1:** have the verify gate skip the fail-on-base check for a task that is not a bug fix, or let the Sketch or plan declare that kind.
- **G2:** reword the finding so it does not tell the agent to invent a bug when the task has none.

**Per change:**
- **C1** traces to G1. It adds `gateslot.NoBugFixedTitle` and the title loop in `exempt` (`internal/gateslot/redfirst.go:26-29`, `internal/loomshed/redfirstgate.go:187-193`). A parent's added entry in the decision record declares the kind, and the gate then skips the check.
- **C2** traces to G2. It rewords the green finding in `redFirstFailure` (`internal/loomshed/redfirstgate.go:258-263`). The finding now says "none of them fails before the change" and names the "invent no bug and keep the tests" way forward.
- **C3** traces to G2. It rewords the green refusal in `redFirstCall.run` (`internal/gatecli/redfirst.go:246-252`). This is the CLI side of the same finding, and its text is the log tail the writer sees.
- **C4** is forced by C2. It renames `noSeamAsk` to `parentRecordAsk(title)` (`internal/loomshed/redfirstgate.go:285-290`) so C2's way forward can name the second title. The other three call sites pass `NoTestSeamTitle`, so their output is unchanged.
- **C5** is forced by C1. The stencil gains an ask-first line (`contracts/stencils/darn/darn-template-write.md:115-116`). Without it, a writer could only learn the new route after a failed attempt.
- **C6** is forced by C1. It updates the docs that state the gate's exemptions:
  - the `darn-recipe.yaml` comment,
  - `loom-discussion-spec.md:137-138`,
  - `docs/overview.md:1109`,
  - the `doc.go` files of `gatecli`, `loomshed` and `shedrecipe`.
- **C7** is forced by C1, C2 and C3. It adds the `No bug fixed` row and `noBugFixedRecord` and updates the green-row assertions in `internal/loomshed/redfirstgate_test.go`. It also updates the green subtest assertion in `internal/gatecli/redfirst_integration_test.go:103-104`.

**Per decision:**
- G1 is built by C1, with C5 and C6 supporting it. The entry offers two options joined by "or"; the diff takes the "declare that kind" one. The parent's added entry lives in the decision record, which is the Sketch.
- G2 is built by C2 and C3, with C4 supporting them.

**Findings:** none. Every hunk traces to G1 or G2 or is an edit a traced change forces. Both goal points are built, and no `Pattern-Departure:` trailer is needed.

**Tool calls:** none.
