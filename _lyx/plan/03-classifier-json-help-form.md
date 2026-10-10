# Card 3 — classifier-json-help-form

**Edit:**
- `internal/fabricengine#readOnlyLyxSegment`
- `internal/fabricengine#IsReadOnlyCommand`
- `internal/fabricengine/refscanner_test.go`
- `internal/websterengine/doc.go`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/fabricengine#lyxCommandWord`

**Intent:**
D3: since 57243019b the global `--json` raises help before any pre-run hook or argument check, so a trailing `--json` is a help form.
`readOnlyLyxSegment` admits command words then `--json` as the last word, and command words then `--help` or `-h` then `--json`, under the same command-word rule as the help form today, bare `lyx --json` included.
A word before the trailing help words that is not a command word still rejects, so `lyx fabric add x --apply --json` and `lyx fabric add --json --apply` are rejected.

Bound, stated in the `IsReadOnlyCommand` doc comment: a command that declares a `--json` of its own, local or persistent on a group, shadows the global flag and runs; the classifier does not know that set, and the walk test of Card 4 pins it, so a new one is a reviewed change against this rule.
Today the shadowing commands are the generic shed `status` verb under each mount, which only reads.

`internal/websterengine/doc.go`'s audit sentence on the classifier and the "audit: fabric-reference" row of `contracts/specs/refusal-spec.md` each add the `--json` help form to the forms Card 2 listed.

Rows of `TestIsReadOnlyCommand`, admitted: `lyx --json`, `lyx fabric add --json`, `lyx fabric --help --json`, `lyx webster record-batch -h --json`.
Rejected: `lyx fabric $v --help --json`, `for v in a b; do lyx fabric $v --help --json; done`, `lyx fabric add x --apply --json`, `lyx fabric add --json --apply`.

**ImpactSummary:** one more trailing-word form in the `lyx` segment rule; no other classifier rule or caller changes.
