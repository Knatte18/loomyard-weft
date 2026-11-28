# Every run profile with a tapestry block gains a jacquard block, carried over from the operator's tapestry values once at reconcile

`jacquard-run-profiles` · Task · feature, profiles, config · running · Code-Burler-Review

Decision record: [Sketch](decision-jacquard-run-profiles.md)

Filed from the Sketch `recipe-build-from-sketch`.

Takes: D7 whole: each profile of internal/runprofile/template.yaml holding a tapestry block gains a jacquard block with the same keys minus loom.discussion, and defaults gains jacquard: solid; a one-time reinterpretation in internal/runprofile beside ReinterpretArrasDefault, registered in internal/configreg's runprofile entry after the arras one, so internal/hubreconcile/walk.go runs it after the template follow unchanged; it gives every profile holding a tapestry block a jacquard block equal to that block minus loom.discussion, and sets defaults.jacquard to the profile defaults.tapestry names when that profile holds a jacquard block, dropping it otherwise. The block key is the literal recipe name jacquard, which needs no other part.
Delivers: Shipped and operator-tuned profiles hold jacquard blocks after the next reconcile, with defaults naming jacquard, so a jacquard run and raddle's profile check find a block.
Exports: Fixed for raddle-names-jacquard: the profiles speed, light, solid and wide hold a jacquard block, and defaults.jacquard is solid in the template.

The part changes these packages:

- internal/runprofile (about 230 changed lines)
- internal/configreg (about 5 changed lines)
