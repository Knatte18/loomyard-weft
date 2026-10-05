---
round: 3
focus:
  - "Go comments and doc.go prose changed in internal/yamlengine (reconcile.go, set.go, doc.go), internal/configengine/doc.go, internal/configcli (configcli.go, doc.go) and internal/boardengine (config.go, doc.go): does any line still hold two independent clauses joined by a semicolon or a comma plus a coordinating conjunction, or a sentence wrapped at a fixed column, against PATTERN-comment-line-breaks?"
  - "Comments added or changed in the same diff outside those files: do any new or changed comment lines break the one-sentence-per-line rule that the sites above are held to?"
---

The review range is unchanged.
The focus targets comment line breaks only, the one rule the previous round's findings cited.
