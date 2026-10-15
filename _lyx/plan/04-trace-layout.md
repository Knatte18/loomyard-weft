# Card 4 — trace-layout

**Edit:**
- `internal/websterengine#buildKickstartPack`
- `internal/websterengine/doc.go`
- `internal/websterengine_test#TestBeginBatch_KickstartPack`

**Intent:** Record each fork prompt's layout on the begin-batch trace line (D5), so a later audit can split forks by layout.

`buildKickstartPack`'s `logPack` closure adds the key `pack_layout` last, after `unresolved`, to its `webster: kickstart pack` Info line: `file` when the state is `on` and `deps.Config.KickstartPackFile` is true, `inline` when the state is `on` and it is false, and `none` for `off`, `language-none` and `empty`.
The existing keys, their order and their values do not change, so existing log pins and the audit scripts' matches on the leading keys hold.
Its doc comment names the new key.

`doc.go`'s trace-line paragraph adds `pack_layout` (`file`, `inline` or `none`) after `unresolved`, with `none` for every line on which no pack was built.

`TestBeginBatch_KickstartPack`'s `wantLog` needles each gain the layout suffix, so every row pins the value on its line:
the `on` rows of the file layout end with ` pack_layout=file`;
"the inline layout carries the pack in the prompt and writes no pack file" with ` pack_layout=inline`;
the off, language-none and empty rows with ` pack_layout=none`;
the two flip rows need both lines, "a flip from inline to file writes the pack file" `batch=1 kickstart_pack=on resolved=1 unresolved=0 pack_layout=inline` and `… pack_layout=file`, and the reverse row the same two in the reverse sense.

**ImpactSummary:** One key appended to one Info line, read only by the audit scripts, whose matches on the leading keys are unchanged; one test's log needles and one doc paragraph follow.

**Commit:** 4: webster trace line records pack_layout
