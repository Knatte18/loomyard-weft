# Card 6 — planparser-join

**Create:**
- `plan:internal/planparser#JoinPart` -> `type JoinPart struct`
- `plan:internal/planparser#JoinedCard` -> `type JoinedCard struct`
- `plan:internal/planparser#JoinResult` -> `type JoinResult struct`
- `plan:internal/planparser#ErrJoinRefused` -> `var ErrJoinRefused = errors.New("planparser: join refused")`
- `plan:internal/planparser#JoinParts` -> `func JoinParts(planDir, title string, parts []JoinPart) (*JoinResult, error)`
- `plan:internal/planparser#ExportMet` -> `func ExportMet(plan *Plan, export string) bool`
- `plan:internal/planparser#TestJoinParts` -> `func TestJoinParts(t *testing.T)`

**Edit:**
- `internal/planparser/doc.go`
- `pattern/PATTERN-sole-parsers.md`

**Extends:**
- `contracts/specs/loom-plan-spec.md`

**Uses:**
- `internal/planparser#moveCardBytes`
- `internal/planparser#rewriteCardIDs`
- `internal/planparser#cardFileName`
- `internal/planparser#ParsePlan`
- `internal/planparser#ParseRef`

**Intent:** Add the join write path (D8), the plan format's one writer of a plan joined from parts, so Go numbers and orders the parts' cards and the chair spends no turns renumbering.

`JoinPart` holds `Name`, `Dir` (the part's plan directory) and `Exports []string` (the part's exports as the parts file spells them).
`JoinParts` parses each part with `ParsePlan`, in the order given, which is the parts file's order, and refuses with an error wrapping `ErrJoinRefused` whose text names the part to fix and the reason, before it writes anything, when:
- two parts' overviews differ in `language:` or `root:`, naming both parts and both values;
- a part's export that `ExportMet` reports unmet over the part, naming the part and the export;
- a `plan:` handle or a new file path is a Create target or Rename New side of cards of two parts, exported or not, naming both parts and the ref.

Otherwise it numbers the parts' cards into one sequence from 1 in part order: each card keeps its slug and moves to its new number with `moveCardBytes`, which rewrites its heading and `Commit:` prefix, and `rewriteCardIDs` rewrites every card id in it that names a card of the same part to that card's new number; its file name is `cardFileName` of the new number and slug.
It then removes every top-level card file and `00-overview.md` the plan directory held, leaving subdirectories such as archives in place, writes every joined card, and writes `00-overview.md` last: frontmatter `format: 5`, `approved: false` and the parts' shared `root:` and `language:`; `# Plan: <title>`; a framing paragraph naming the parts in order, which the chair rewrites; a `## Card Index` of every card's new line; a `## Shared Decisions` section holding each part's own section body under `### Part <name>` for the parts that have one; and the canonical `## Rename mechanic` section when any part has one.
`JoinResult.Cards` lists one `JoinedCard{Part, From, To, File}` per card, in the new order.
A read or write failure is a returned error that does not wrap `ErrJoinRefused`, and it names the way forward: run `lym loom join-parts` again, which rewrites the whole plan directory from the parts, since a join that failed wrote no complete plan.

`ExportMet` reports whether a card of the plan creates the export (a Create target or Rename New side), re-signs it (an Edit arrow target) or deletes it (a Delete target): a path by its disk path, a glyph exactly, and a `plan:` handle exactly or by the declaration it names, a handle whose body `ParseRef` reads with the same `Unit` and `Name` as the export's.
The resolve pass canonicalizes a declared handle from its draft unit and its declaration head, keeping the unit and the declared name and spelling the owner chain from the head, so the canonical spelling it writes on disk still meets an export the parts file spells as drafted.
The part check (D7) applies the same test, which is why it is exported.

The package doc's list of write paths, and `pattern/PATTERN-sole-parsers.md`'s plan-format list of write paths, gain `JoinParts`.
`contracts/specs/loom-plan-spec.md` gains a `## Joined plans` section stating what `JoinParts` refuses and writes.

`TestJoinParts` (tier0) builds part directories with `internal/testkit/plankit`: three parts whose second card of one part names its first card by card id join into one plan numbered in part order, with headings, `Commit:` prefixes and the card id rewritten, which `ParsePlan` and `ValidateFormat` accept with no format finding, and stale card files in the plan directory are gone while an archive subdirectory stays; a part exporting a member it re-signs with an Edit arrow, another exporting a file it deletes, and a third exporting `plan:<pkg>#Foo` whose card declares the owner-qualified `plan:<pkg>#T.Foo` join with no refusal; a part with a `## Shared Decisions` section and a part with a `## Rename mechanic` section give the joined overview the first under `### Part <name>` and the canonical rename mechanic; then one row per refusal above, each asserting `errors.Is(err, ErrJoinRefused)`, the named part, and that the plan directory is unchanged.

**ImpactSummary:** One new write path in planparser; no existing reader or writer changes, and the PATTERN background lists it.
