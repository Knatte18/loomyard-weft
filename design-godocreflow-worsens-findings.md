# tools/godocreflow worsens wrapped-break findings instead of clearing them

`godocreflow-worsens-findings` · Task · bug, tools, lint · running · Drift-Await

Imported from [issue #183](https://github.com/Knatte18/loomyard/issues/183).

What happened: `tools/godocreflow` was run over trimmed Go files to clear 624 wrapped-break findings from `lym loom lint-comments`.
Findings rose to 916.
It skips whole comment blocks that hold a heading or list, and it joins lines past the length limit.
The commit was dropped and the findings were fixed by hand.

Why abnormal: the reflow tool is meant to fix these findings and produced more of them.

What to do: make godocreflow process blocks with headings or lists, and make it respect the line length limit, or stop pointing at it as a fix.

Provenance: task doc-go-trim-pass; note Darn-Write.md.