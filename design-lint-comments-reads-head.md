# lym loom lint-comments --base checks committed HEAD, not the working tree

`lint-comments-reads-head` · Task · bug, lint, loom · running · Code-Burler-Review

Imported from [issue #182](https://github.com/Knatte18/loomyard/issues/182).

What happened: `lym loom lint-comments --base` reads committed HEAD, not the working tree.
A fix to a flagged comment could be verified only after committing it.

Why abnormal: a lint run during editing gives no feedback on uncommitted changes, which forces a commit per check cycle.

What to do: lint the working tree, or add a flag that does, or document the HEAD-only behaviour in the verb's help.

Provenance: task doc-go-trim-pass; note Darn-Write.md.