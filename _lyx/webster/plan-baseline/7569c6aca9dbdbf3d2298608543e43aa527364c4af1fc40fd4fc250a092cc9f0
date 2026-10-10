# Card 10 — verify-chain-splitter

**Create:**
- `plan:internal/planparser#chainSegment` -> `type chainSegment struct`
- `plan:internal/planparser#splitChain` -> `func splitChain(chain string) []chainSegment`

**Edit:**
- `internal/planparser#crossModuleCommands`
- `internal/planparser#moduleWideCommands`
- `internal/planparser/validate_test.go`
- `internal/planparser/doc.go`
- `//contracts/specs/loom-plan-spec.md`

**Intent:** Give `verify-nested-module` and `verify-module-wide` one chain splitter so both see a `go` command after `||` and `|` (D5).
`splitChain` joins a backslash-newline continuation into one line first, then splits at `&&`, `||`, `;`, `|` and a newline, reading `||` as one separator and never as two pipes, and reading a separator inside a single- or double-quoted span as text.
Each `chainSegment` carries the segment text and the separator that preceded it, so a caller can tell a sequencing separator from a pipeline stage.
`moduleWideCommands` and `crossModuleCommands` iterate the splitter's segments in place of their own `strings.NewReplacer` splits.
In `crossModuleCommands` a `cd` moves the run directory only when its segment follows `&&`, `;` or a newline; a `cd` in a pipeline stage or after `||` ends the scan, as an unresolvable `cd` does today.
`TestValidate_VerifyNestedModule` and `TestValidate_VerifyModuleWide` gain the acceptance rows: a `go` command after `||` and after `|` is found, `a || b` is one separator, a separator inside quotes is text, `go test \` continued on the next line is one command, and a `cd` in a pipeline stage or after `||` ends the cross-module scan.
`doc.go` and the two spec rows name the separators the checks split at and the continuation rule.

**ImpactSummary:** Two planparser scanners share one new splitter; no exported signature changes and both plan gates pick the wider split up through ValidateFormat.
