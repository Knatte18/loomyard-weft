---
round: 1
focus:
  - "internal/secretpath/secretpath.go: does the built-in deny-list match the entry's named patterns (.env, *.pem, id_rsa, credentials*, secrets.y*ml) at any directory depth, and does Match handle case, leading dirs and look-alike names correctly?"
  - "internal/fabricengine/commit.go and internal/websterengine/recordbatch.go: does each guard refuse only paths the commit newly adds, and does a refused fabric commit leave no commit and no partial state?"
  - "internal/gitrepo/gitrepo.go NewPathsIn: does it list added paths correctly for the first commit, merge commits, renames and multi-commit batches, and does it fail closed on a git error?"
  - "internal/fabricengine/config.go and template.yaml: does the hub-added pattern config merge with the shipped list rather than replace it, and does a malformed pattern surface as an error instead of being skipped?"
  - "Scope against the decision record's Goal: does every changed file serve the record-batch and fabric commit guards plus hub-added patterns, with the pattern/ entries and docs landing in the same diff?"
---

Decision record holds only the board entry under `## Goal`, so the measuring stick is that entry.
