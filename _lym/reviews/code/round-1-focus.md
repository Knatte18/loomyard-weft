---
round: 1
focus:
  - "internal/boardengine/store.go `promoteTo`: does a plain promote of a discussed note with a stored record reach the raddle path for slug and handle input, and does a Sketch given `--recipe` of any other value still hit the refusal in `checkDiscussedPromotion` naming the plain promote?"
  - "internal/boardengine/store.go `promoteTo`: a discussed note whose record is not stored, or a note that is not discussed, must keep today's recipe handling on a plain promote; check that `recordHeld` does not change their result."
  - "Refusal texts built from `SketchPromoteCommand` in boardengine/sketch.go, labels.go and battencli/startguard.go: is the quoted JSON-slug form accepted by the verb as written, and does the text stay correct when the refusal is reached from a path other than promote, such as upsert?"
  - "Test coverage in boardengine/sketch_test.go, labels_test.go and battencli/startguard_test.go against the Wanted list: plain promote by slug, by handle, `--recipe raddle` accepted, another recipe refused; name any Wanted behavior left uncovered and any test that duplicates another."
  - "Docs and skill the Wanted list names (docs/overview.md, boardcli/cli.go, boardcli/doc.go, boardengine/doc.go, plugins/ly/skills/board/SKILL.md): does any text still call `--recipe raddle` the required form, and does the new `ly:board` text match what the code does?"
---

Seed pass: the range is one commit, so focus targets the promotion rule in `promoteTo`, the shared refusal command and the Wanted list's doc updates.
