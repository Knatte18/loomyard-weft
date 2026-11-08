---
round: 2
focus:
  - "In internal/fabricengine tests removed over the review range, check whether any dropped test asserted an archive tag (the archive/<slug>/<tip> name, tags present or absent on origin or locally, or tag reuse) without a kept test pinning the same assertion; the record's keep rule requires such a test to carry a testtiming:keep reason."
  - "Check that every restored or folded archive-tag assertion carries a testtiming:keep line naming the tag it pins, and that any helper it needs (such as archiveTagsAt or showAtTag) exists so the package still builds."
---

The focus follows the one round 1 finding and asks only where to look.
