# Plan-review rubric lets a reviewer recommend a card retyping that validate-plan rejects

`plan-rubric-containment` · Note · bug

Imported from [issue #438](https://github.com/Knatte18/loomyard/issues/438).

A plan-review round (Burler, plan segment) raised a finding asking for symbol renames in two cards to be typed `Rename` over member glyphs, reading only the spec's type table.
The author's fix attempt then failed `lyx loom validate-plan` with blocking `containment-file-overlap` findings: other cards target the files holding those symbols whole, so the recommended retyping cannot exist in that plan's file-level layout.

Why this is abnormal: the review rubric's `Do not flag` list covers checks the gate already runs, but not the interaction between a card type and the cross-card containment check.
The reviewer therefore issued a remedy the gate cannot accept, and the review round was spent on an unfixable finding.

What to do: have the plan-review rubric tell the reviewer to run `lyx loom validate-plan` before recommending a card retyping, or add the type-vs-`containment-file-overlap` interaction to the `Do not flag` list.

Provenance: task `fabric-pushes`, note `burler-plan-r1.md`.
