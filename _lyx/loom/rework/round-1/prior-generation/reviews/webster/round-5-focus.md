---
round: 5
focus:
  - "Sweep every Go comment line added or changed in the review range, across all packages and all test files, not just a sample, and settle whether each comment follows `PATTERN-comment-line-breaks` in both directions: no sentence breaks inside a clause, and no line holds two independent clauses (a comma splice, a semicolon join, or a clause opened by `so` with its own subject). Include lines changed only by reindentation. Sites touched by earlier fixes in boardcli, configcli, configsync, selfreportcli and yamlengine are in scope again, since their neighbouring blocks have repeatedly held the same fault."
  - "List the changed comment lines per file before judging, so that a file with changed comments and no finding is visibly checked. Settle whether any package outside boardcli, configcli, configsync, selfreportcli, yamlengine, boardengine and configengine has changed comments with the same fault."
---

The same fault has been raised at new sites in each of rounds 2 to 4, so an exhaustive enumeration, not a spot check, is what settles it.
