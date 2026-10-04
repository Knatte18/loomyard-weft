# fabric: two-sided reset-to-SHA verb

`fabric-reset-to-sha` · Notes · feature

From `manifest/roadmap.md` Someday.
The post-conclude undo the merge surface deliberately does not ship: `MergeAbort` covers only the uncommitted merge-attempt window, so a landed merge is final at the Fabric layer.
Closing it means a reset to a visible warp SHA that resolves the paired weft SHA through the correspondence index and routes both sides through the destruction gate.
See the `internal/fabricengine` package documentation's merge section.