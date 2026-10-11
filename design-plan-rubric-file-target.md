# Plan review rubric gives no warning about the redundant-file-target check

`plan-rubric-file-target` · Note · enhancement

Imported from [issue #542](https://github.com/Knatte18/loomyard/issues/542).

In a fixing-plan round, a review finding proposed adding whole-file targets beside member glyphs in the same files.
`lyx loom validate-plan` refuses that shape with its `redundant-file-target` check, so the proposed fix broke the rubric's "Proposed fixes" rule.
The reviewer could not see this without running `lyx loom validate-plan` on a trial edit, which cost an extra validate round trip.
The fix fell back on the check's own way forward: target the whole file in place of the member.

Why abnormal: the reviewer rubric tells reviewers how to propose fixes but does not state the `redundant-file-target` constraint, so a rubric-following proposal can still be refused by the gate.

What to do: state the `redundant-file-target` rule in the rubric's "Proposed fixes" section (a file target and a member glyph in the same file are mutually exclusive), or have the reviewer prompt point at `lyx loom validate-plan` as a pre-check for proposed target changes.

Provenance: task `twill-discusser`; notes: burler-plan-r1.md
