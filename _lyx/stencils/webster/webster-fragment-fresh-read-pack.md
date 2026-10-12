<!-- This is the pack-on FRESH-READ rule for webster's implementer prompt.
     RenderForkPrompt reads it through stencilstore when begin-batch built a kick-start pack;
     the text, with this banner and the trailing newline stripped, fills the required fresh_read marker of webster-body-implementer.
     It carries no marker.
lyx-stencil: sha256=245af197c251fbf52f2dcdbbe3e17a5f4b16ee8bf5ad18cf4917aa4635d8f29e build=1135939906fcdddd4cfdcf1bd7b46ad7ace09268 time=2026-10-12T11:39:01Z -->

Inherited context can be stale.
A file you looked at during orientation — your own, or one Master read before forking you — is not necessarily the version on disk right now: a prior batch's own card commits may have changed it since.
The one turn at each card's start (step 1) is that card's fresh read: it re-reads the card file and the listed spans, not the whole files that hold them.
In the turn right after it, before any edit, read whole every file the card's own target list and `Uses:` list name that no listed span covers, such as a plain path or a self glyph's file.
An unresolved member glyph's file is read whole in the turn right after the `lyx quarry resolve` call or grep that locates it, still before any edit.
For a file an earlier card of this batch edited, the fresh read is step 1's whole-file read in that turn.
Only your own reads, taken now, are current;
content you merely inherited is not.
