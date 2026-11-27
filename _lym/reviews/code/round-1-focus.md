---
round: 1
focus:
  - "internal/desktoplauncher: on every launcher form, which text does a user see as the launcher's name (Name= key, file name, rem line), and does each form meet the board entry's \"lym: <shortname>\"?"
  - "_lym/landing/summary.md: check each claim against the code, in particular where the Name= and rem text shows, and that a standing launcher is rewritten in place on a later dobby start (follow Desktop.Add and its callers)."
  - "internal/desktoplauncher Desktop.content and cmdFile: do the changed doc comments and doc.go describe what each form writes, for the all-hubs launcher too, including which line \"title\" appears on versus the Comment= key?"
  - "Does any reader, test, doc or sandbox suite outside the diff still depend on or state the old \"start the hub <shortname>\" name?"
---

The diff is small: one argument change and a parameter rename in `internal/desktoplauncher`, with matching test rows and a package doc line.
The questions above are where its effect reaches beyond the changed lines.
