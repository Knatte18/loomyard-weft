# Card 9 — The card-fabric-reference check in planparser

**Create:**
- `internal/planparser/fabricref.go`
- `internal/planparser/fabricref_test.go`

**Edit:**
- `internal/planparser/plan.go`
- `internal/planparser/parse.go`
- `internal/planparser/doc.go`

**Intent:**
Implement D13's check as a pure planparser function, wired into the gates by card 10.

Model: `Plan` gains the overview file's raw text and `Card` gains its card file's raw text, filled by `ParsePlanFrom` from the bytes it already reads, so the check reads no file itself and `planparser` stays the sole parser (`PATTERN-sole-parsers`).

`fabricref.go` declares the narrow seam `planparser` receives the rule through — an interface with the two methods card 8's location-free rule has, path rule and spelling rule, each returning the matched text and whether it matched — so `planparser` imports neither the fabric side nor `weftname`.
It declares the check `card-fabric-reference` as an exported function taking the plan and that matcher and returning `[]ValidationError`; like `CheckFirstCard` it sits outside `ValidateFormat` and `Validate`, since only callers holding a matcher can run it.
Spans scanned, each passed to the matcher on its own so quote state never carries between spans:
- a card's `**Verify:**` value and the overview's `## verify:` section body: path rule and spelling rule;
- every fenced code block of every plan file, cards and overview alike, whatever its info string: path rule and spelling rule;
- every inline code span of every plan file: path rule only, because a span documenting a command opens with the spelling.
Prose outside code is never scanned.
A finding names the card (or, for a hit in the overview, no card and the overview section the hit is in), and the matched text, and its way forward: take the data from committed test data an earlier card creates, or drop the command.
Every comment and string here says "fabric repo", never the side's name (`PATTERN-fabric-vocabulary`).
State the bound in the check's comment: it only refuses and removes no guard; it misses a spelling in `bash -c "…"` or built from a variable, the rest of a span after an unterminated quote for the bare-word and spelling patterns, a spelling in an inline span or prose, and a fabric-repo path not spelled as a name ending in the suffix; the implementer audit stays the guard for those.

`doc.go` adds the check to the package's check list, outside both entry points, beside `CheckFirstCard`.

Tests (`fabricref_test.go`, an external test package so it may build the real rule from `internal/fabricengine`): build sibling paths with `fabricengine.WeftWorktree` over a synthetic location, as `internal/websterengine`'s audit tests do, and spellings from `lyx fabric …` or one of the three exempt spellings (`lyx weft sync`, `lyx warp checkout`, `lyx.exe weft push`); a table over a plan built in a temp directory: a sibling path in a card Verify, the overview verify section, a fenced block and an inline span each fails naming the card or the overview section and the matched text; a `lyx fabric` spelling fails in the first three and passes in an inline span and in prose; the same plan without them passes; a quoted search pattern in a fenced block passes.

**ImpactSummary:** planparser's model gains two raw-text fields that only the new check reads; parsing and every existing check are unchanged.
