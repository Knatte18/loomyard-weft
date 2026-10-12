---
round: 5
focus:
  - "Decision D4 in `_lyx/discussion/decision-record.md`: its category test says a key is denied to the local layer when it switches off or hollows a guard, including a review or the lens set a reviewer is given. Apply that test as written to every key of every module's `template.yaml`, including keys whose value selects a mode that changes whether a review runs, such as `loom.discussion_producer` and its twill fan in `internal/loomengine/twill.go`. Settle whether every key left in `LocalKeys` is bounded by the stated Bound and by R3, and whether `LocalDenied` and the acceptance row listing its examples agree with the test."
  - "Decision D1 and Decision D6 in `_lyx/discussion/decision-record.md`: layers are stacked from two roots, the repo default under `<BoardDir>/_lyx/config/` and the worktree and local files under the anchor, while `configengine.load` expands `${env:...}` markers with `envsource.Build(baseDir)`. Settle whether the record decides which `.env` expands a marker in a stacked config, and whether D6's claim that the strip never changes a held key's resolution follows from that decision."
  - "The strip's acceptance rows in `_lyx/discussion/decision-record.md`: D6 compares unexpanded nodes and rules that a marker equals only the same marker text. Settle whether the rows pin both the literal-over-marker and the marker-over-literal case."
---

The round-4 judge found the D4 bound finding recurring for the fourth round through a new key each time.
The first entry asks the reviewer to apply the category test to the whole key set, not to spot-check keys already named.
