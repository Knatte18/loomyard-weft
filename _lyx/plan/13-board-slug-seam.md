# Card 13 — board-slug-seam

**Edit:**
- `internal/boardengine#Config`
- `internal/boardengine#Board`
- `internal/boardengine#New`
- `internal/boardengine#Board.boardCriticalSection`

**Create:**
- `plan:internal/boardengine#checkNewSlugs` -> `func checkNewSlugs(loaded, after []Task, renamedTo string, validate func(slug string) error) error`
- `plan:internal/boardengine_test#TestSlugRule` -> `func TestSlugRule(t *testing.T)`

**Extends:**
- `internal/boardengine/doc.go`

**Uses:**
- `plan:internal/boardengine_test#newProjectBoard`

**Intent:**
D14: the board applies fabric's pair slug rule through a seam the CLI fills (card 17).
`Config.ValidateSlug func(slug string) error`, copied onto `Board`; nil checks nothing, which is what the detached sync, a batten claim and the owner setter build, none of which makes a slug.
`checkNewSlugs` runs in `boardCriticalSection` after the mutation and judges only a write that makes a task or project slug: an entry of kind task or project absent from the loaded entries (a new upsert, batch or merge entry), an entry whose kind became task or project from note (`lyx board promote`), and the target of a rename (`renamedTo`, from the `RenameResult`).
Its refusal names the entry, the rule's own reason (the seam's error text) and a slug the rule admits as the way forward, the slug with its offending suffix dropped or `-task` appended to a reserved name.
`checkNewSlugs` runs that suggestion through the `validate` seam and against the slugs of the after entries before it offers it, and offers none when the suggestion fails the rule or names an existing entry, since an upsert under an existing slug would edit that entry instead of creating one; the refusal then names the reason and, as the way forward, choosing another slug the rule admits.
A write that leaves an entry's slug and kind as they were is never refused, so an existing slug the rule now refuses keeps taking writes; a kind change from task to project keeps a slug that already names a pair and is not judged, so `lyx beam promote` (through the owner setter, which runs outside this section) checks none.
`TestSlugRule` drives a `Board` from `newProjectBoard`, setting a `ValidateSlug` fake through its `configure` hook that refuses a slug ending in the records suffix and the name `_board`: a new task, a new project, a promote of a note and a rename to such a slug are each refused naming the reason and the admitted slug (`x` for `x` with the records suffix appended, `_board-task` for `_board`), a new task whose suggestion collides with an existing entry is refused naming the reason and choosing another slug, offering none, a body edit of an existing such task passes, and a note created with such a slug passes.

**ImpactSummary:** One optional seam on the board's write section; nil, the default for every existing Board, refuses nothing.
