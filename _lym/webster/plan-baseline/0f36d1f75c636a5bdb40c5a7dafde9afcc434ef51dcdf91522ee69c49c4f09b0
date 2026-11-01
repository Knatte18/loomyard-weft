# Card 3 — parts-parser

**Create:**
- `internal/partsparser/doc.go`
- `plan:internal/partsparser#DirName` -> `const DirName = "parts"`
- `plan:internal/partsparser#Dir` -> `func Dir(anchorPath string) string`
- `plan:internal/partsparser#DirRel` -> `func DirRel() string`
- `plan:internal/partsparser#File` -> `func File(anchorPath string) string`
- `plan:internal/partsparser#PartDir` -> `func PartDir(anchorPath, part string) string`
- `plan:internal/partsparser#PartOverview` -> `func PartOverview(anchorPath, part string) string`
- `plan:internal/partsparser#JoinRecord` -> `func JoinRecord(anchorPath string) string`
- `plan:internal/partsparser#ArchivePrefix` -> `const ArchivePrefix = "archive-"`
- `plan:internal/partsparser#Parts` -> `type Parts struct`
- `plan:internal/partsparser#Part` -> `type Part struct`
- `plan:internal/partsparser#SharedSpot` -> `type SharedSpot struct`
- `plan:internal/partsparser#Parts.Part` -> `func (p *Parts) Part(name string) (Part, bool)`
- `plan:internal/partsparser#Parts.External` -> `func (p *Parts) External(name string) []string`
- `plan:internal/partsparser#Parts.SpotPaths` -> `func (p *Parts) SpotPaths() []string`
- `plan:internal/partsparser#Parse` -> `func Parse(data []byte) (*Parts, error)`
- `plan:internal/partsparser#Finding` -> `type Finding struct`
- `plan:internal/partsparser#Check` -> `func Check(parts *Parts, decisionIDs []string, max int) []Finding`
- `plan:internal/partsparser#Validate` -> `func Validate(anchorPath, recordPath string, max int) ([]Finding, error)`
- `plan:internal/partsparser#TestParse` -> `func TestParse(t *testing.T)`
- `plan:internal/partsparser#TestCheck` -> `func TestCheck(t *testing.T)`
- `plan:internal/partsparser#TestValidate` -> `func TestValidate(t *testing.T)`
- `contracts/specs/loom-parts-spec.md`
- `plan:contracts/specs#LoomPartsSpec` -> `var LoomPartsSpec []byte`

**Edit:**
- `PATTERN.md`
- `pattern/PATTERN-sole-parsers.md`
- `contracts/specs#TestRegistry_NamesAndDefaults`

**Extends:**
- `contracts/specs#entries`

**Uses:**
- `plan:internal/discussionparser#DecisionIDs`
- `internal/seatengine#ValidateSeatName`
- `internal/lymdirs#LymDirName`

**Intent:** Add `internal/partsparser`, the parts file's sole parser (D3), with its format pinned in a new spec.

**Paths.** `_lym/parts/` is tracked durable state (PATTERN-durable-vs-ephemeral-state), named only here, joined from `lymdirs.LymDirName` and `DirName`, never a `_lym` literal: `Dir` is `<anchor>/_lym/parts`, `DirRel` the forward-slash `_lym/parts`, `File` is `parts.md` in it, `PartDir` is `<Dir>/<part>`, `PartOverview` is `00-overview.md` in the part directory, and `JoinRecord` is `join.md` in `Dir`.
`ArchivePrefix` starts the name of every archive directory under `Dir`.

**Model.** `Parts` holds `Title`, `SplitNote`, `Parts []Part` in file order and `Spots []SharedSpot`.
`Part` holds `Name`, `Decisions []string`, `TakesIn`, `Delivers`, `Exports []string`, `DependsOn []string` and `Area []string`.
`SharedSpot` holds `Path`, `Decisions []string`, `NeededBy []string`, `Needs`, `Exports []string` and `Uses []string`.
`Parts.Part` returns the part named name.
`Parts.External` returns the exports another part's cards may use from the part named name: every export of each part it depends on, then every export of each shared spot whose `Needed by` names it, deduplicated, in that order.
`Parts.SpotPaths` returns every shared spot's path.

**Format**, which `Parse` reads and the spec pins:

````markdown
# Parts: <task title>

## Split note

<how the task splits, or why it does not>

## Part: <name>

- Decisions: D1, D4
- Takes in: <prose>
- Delivers: <prose>
- Exports:
  - `plan:internal/x#NewThing`
  - `internal/x/newfile.go`
- Depends on: <name>, <name>
- Area:
  - `internal/x`
  - `contracts/specs/x-spec.md`

## Shared spot: `<path>`

- Decisions: D3
- Needed by: <name>, <name>
- Needs: <prose>
- Exports:
  - `plan:internal/y#Registered`
- Uses:
  - `plan:internal/x#NewThing`
````

The H1 is `# Parts: <title>`; then one `## Split note`, the `## Part: <name>` sections and the `## Shared spot: `<path>`` sections, in that order.
A field is a top-level bullet `- <Label>: <value>`; `Decisions`, `Needed by` and `Depends on` hold comma-separated values, `Takes in`, `Delivers` and `Needs` hold prose that may continue on indented lines, and `Exports`, `Area` and `Uses` hold indented sub-bullets of one backticked ref each.
A part requires `Decisions`, `Takes in`, `Delivers` and `Area`; a shared spot requires `Needed by` and `Needs`; every other field is optional and is left out when empty, never written as `none`.
Lines inside fenced blocks are read as text.
`Parse` returns an error naming the line and what is wrong, with the way forward to correct that line as `contracts/specs/loom-parts-spec.md` shows, for: a missing H1 or split note, an unknown section heading or field label, a required field missing or empty, a ref sub-bullet that is not one backticked ref, and a field repeated in one section.

**Check** returns one `Finding{Check, Part, Detail}` per violation, `Part` naming the part or the spot's path, each `Detail` ending in its way forward:
- `parts-none` — no part; write at least one part, a single part covering the whole task when it does not split.
- `parts-over-max` — more parts than max; merge parts until at most max remain.
- `part-name-invalid` — a name `seatengine.ValidateSeatName` refuses; rename the part to a lower-case name that starts with a letter.
- `part-name-repeated` — a name an earlier part holds; rename one of them.
- `part-name-archive` — a name beginning with `ArchivePrefix`, whose directory the preparations would skip as an earlier archive; rename the part.
- `part-dependency-order` — a `Depends on` name that is no earlier part (later, unknown or the part itself), so file order is a topological order; move the depended-on part earlier or drop the dependency.
- `spot-needer-unknown` — a `Needed by` name that is no part; name a part of the file.
- `spot-export-order` — a spot whose `Uses` holds an export of a part that comes after the first part its `Needed by` names, since no card position satisfies both; reorder the parts or move the dependency into a part. An export of that first part itself, or of an earlier part, passes.
- `export-claimed-twice` — a ref two parts, two spots, or a part and a spot both export; keep it on one of them.
- `spot-in-area` — a spot path a part's `Area` lists as a file entry of its own; drop it from the area. A spot inside a package directory an area names passes.
- `decision-unnamed` — a decision id of the record no part or spot names; name it in the part or spot that carries it.
- `decision-unknown` — a named id the record does not hold; drop or correct it.

**Validate** is the one function the `parts` gate and `lym loom validate-parts` call: it reads `File(anchorPath)`, parses it, reads the record at recordPath and checks the parts against `discussionparser.DecisionIDs` of it.
An absent parts file is a `parts-missing` finding whose way forward is to write it, a `Parse` error is a `parts-malformed` finding carrying the error, and any read error, an absent record included, is a returned error: the record is Discussion-Write's committed output, which the Parts-Write agent cannot write, so an absent one ends the gate at once with the error, naming `lym loom validate-discussion` for the parent, instead of spending its re-prompts.

**Spec.** `contracts/specs/loom-parts-spec.md` opens with the status line used by `loom-plan-spec.md` (a pinned Go-to-Go contract kept under `contracts/specs/`), names `Parts-Write` as producer and `Parts-Plan-Write` as consumer, and pins the format and every check above with its way forward, plus the paths: the parts file, each part's directory holding a plan in `loom-plan-spec.md`'s format, the join record, and archive directories under the `archive-` prefix.
It says the LLM-facing subset lives in the `loom-template-parts` stencil.
It registers as `loom-parts-spec`, embedded as `LoomPartsSpec` beside `LoomPlanSpec` with an entry in `entries`; `TestRegistry_NamesAndDefaults` adds it to its expected names.

**PATTERN.** The declared change of `PATTERN-sole-parsers` lands here; its overview line becomes:

```markdown
- `PATTERN-sole-parsers` — Parsing plan, parts file, discussion, summary, recipe or halt note, or classifying a plan ref: only its sole parser. (test) — [background](pattern/PATTERN-sole-parsers.md)
```

`pattern/PATTERN-sole-parsers.md` gains a `## Parts format (`internal/partsparser`)` section: `internal/partsparser` is the sole parser of `_lym/parts/parts.md` and the sole declarer of the parts directory's paths; it reads decision ids only through `discussionparser.DecisionIDs`; a part directory holds a plan in the plan format, which only `internal/planparser` reads and writes.

The package doc names it the parts file's sole parser, its path ownership and that it reads the record's ids through `internal/discussionparser`.
`TestParse` (tier0) covers a full file with two parts and two spots parsed into the model, `External` and `SpotPaths` over it, and one row per `Parse` error above naming its line.
`TestCheck` (tier0) covers a valid file, including a spot inside a package a part's area names and a spot using an export of the first part needing it, with no finding, and one row per check above reaching it.
`TestValidate` (tier0) covers a valid file and record in a temp anchor, the `parts-missing` and `parts-malformed` findings, an absent record returned as an error naming `lym loom validate-discussion`, and a check finding passed through.

**ImpactSummary:** New package and spec; the PATTERN line and its background gain the parts file, and the specs registry gains one entry.
