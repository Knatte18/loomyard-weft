# raddle: each filed part carries its own module labels, not the Sketch's whole set

`raddle-part-labels` · Task · bug, loom, board, stencil · running · Darn-Write

## Seen (operator, 2026-11-17)

Every task raddle files carries all of its Sketch's module labels.
`Raddle-File` builds each entry's labels as its type label plus `entry.ModuleLabels`, the Sketch's whole module set
(`internal/loomshed/raddlefile.go`, the `labels` closure: "Every entry carries its type label and the Sketch's module labels").
So each member of `adopt-build-resplit` carries fourteen module labels, and each `codexengine` member the same four,
whatever packages the part changes.

The board policy asks for a label for each module an entry touches.
A label filter such as `--label reed` then matches parts that never touch reed, and the board listing runs very wide.

## Idea

- The split proposal names each part's module labels, the modules that part's work changes,
  and `Raddle-File` writes those instead of the Sketch's whole set.
- `raddleparser`'s check refuses a part with no module label, and a part label outside the board's vocabulary.
  It accepts a part label that is not among the Sketch's, since a split can find a module the Sketch's labels missed.
- The split perch weighs each part's labels against the packages its body names.
- Raddle's stencil states the rule beside the other part fields.
