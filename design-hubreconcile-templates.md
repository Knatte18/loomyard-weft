# internal/hubreconcile fixtures onto registered templates and CopyHub, with the held-lock test at tier 0

`hubreconcile-templates` · Task · enhancement, fabric, testkit, git · running · Drift-Baseline

Decision record: [Sketch](decision-hubreconcile-templates.md)

Filed from the Sketch `fabric-test-speed`.

Takes: hubforge-registered-templates' registration
Delivers: newStaleHub with pair-a built by a registration, the package's hubforge.NewHub sites on hubforge.CopyHub per D5's rule, and TestEnsure_HeldLockTimesOutWithoutStamping rewritten onto in-memory inputs at tier 0
Exports: nothing

The part changes these packages:

- internal/hubreconcile (about 120 changed lines)
