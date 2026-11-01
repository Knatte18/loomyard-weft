# Card 5 — module-bolt-dir-sweep

**Edit:**
- `//cmd/lym/stencilseed.go`
- `internal/battencli/cli.go`
- `internal/battencli/refusal.go`
- `internal/boardcli/cli.go`
- `internal/boardcli/doc.go`
- `internal/boardengine/doc.go`
- `internal/configcli/configcli.go`
- `internal/configcli/doc.go`
- `internal/configcli/hubwide.go`
- `internal/creelengine/geometry.go`
- `internal/fabriccli/doc.go`
- `internal/fabriccli/fabric.go`
- `internal/hubforge/hub.go`
- `internal/landingshed/doc.go`
- `internal/preflight/doc.go`
- `internal/preflight/predicates.go`
- `internal/reedcli/up.go`
- `internal/reedengine/lifecycle.go`
- `internal/reedengine/serverlog.go`
- `internal/stencilstore/doc.go`
- `internal/webstercli/wiring.go`

**Uses:**
- `plan:internal/fabricengine#BoltDirName`
- `internal/creelengine/doc.go`
- `internal/hubforge/doc.go`
- `internal/reedcli/doc.go`
- `internal/reedengine/doc.go`
- `internal/webstercli/doc.go`

**Intent:** Bring every production message and comment outside fabric that spells the Bolt directory in line with D9, with no behaviour change.

- A message or help text that names the directory takes it from `fabricengine.BoltDirName`: `configcli`'s "committed in _board" results and refusals (`hubwide.go`, `configcli.go`, including the cobra `Long` texts, which concatenate the constant as `fabriccli`'s `Long` already does with `weftname.Suffix`), `fabriccli/fabric.go`'s clone and config help texts, `battencli/cli.go`'s help text, `boardcli/cli.go`'s help text and `reedcli/up.go`'s help text.
  Where the sentence reads better without the name, it says "the Bolt checkout" instead, which also satisfies D9.
- Every comment that spells the path (`<hub>/_board`, `_board/.lym/...`, `_board/_lym/...`, `committed in _board`) names "the Bolt checkout" (or "the Bolt checkout's `_lym/config`" and the like) instead, in each listed file and doc comment.
- The help texts' wording changes nowhere a test pins it with the literal; the tests that did spell it move to the constant in card 6, whose assertions compare the same strings.
- `boardengine/doc.go`'s replay paragraph is not reworded here; card 12 rewrites it.

**ImpactSummary:** Comment, help-text and message spelling only across 21 files in about a dozen packages; every printed value stays the same string.
