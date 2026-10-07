---
round: 4
focus:
  - "Go comment line breaks across the whole diff: scan every added or changed comment line in every touched Go file, including the comments the latest fix commits rewrote. Settle whether each line holds one sentence, whether any line ends mid-phrase without punctuation, and whether any line keeps two independent clauses (a semicolon, or a comma plus and/but/so with its own subject and verb) on one line. A comma before a conjunction that only extends a list or a compound predicate with a shared subject is not a violation. Legacy fixed-column comment lines the diff does not touch are not in scope."
  - "Untagged Go tests the diff adds, renames or edits by more than one line, in every touched package (at least internal/orchengine, internal/loomcli, internal/shuttleengine, internal/shuttleengine/claudeengine, internal/shedadapters, internal/burlerengine, internal/webstercli and internal/websterengine): settle whether each calls t.Parallel as its first statement (and each subtest with its own fixture does too), or carries a comment at the test or atop its file naming the process-global state (environment variable, working directory, logger, package-level variable) that keeps it serial. A test the diff touches by one line is not in scope."
---

Both entries are sweeps: a fixer who patches only the sites a review lists leaves others behind, so the reviewer checks every touched file against the same rule.
