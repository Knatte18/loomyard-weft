# Card 10 — board-membership

**Edit:**
- `internal/boardengine#Config`
- `internal/boardengine#Board`
- `internal/boardengine#New`
- `internal/boardengine#Store`
- `internal/boardengine#Board.boardCriticalSection`
- `plan:internal/boardengine#checkProjectRules`
- `internal/boardengine#UpsertPayloadKeys`
- `internal/boardengine#Store.Rename`
- `internal/boardengine#renameSlug`
- `internal/boardengine#Store.Prune`
- `internal/boardengine#Store.MergeTasks`
- `internal/boardengine#scopeChanged`
- `plan:internal/boardengine_test#TestProjectRules`
- `internal/boardengine_test#TestRename_Refusals`
- `internal/boardengine_test#TestRename_KeepsEntryRewritesDependentsAndReportsMentions`
- `internal/boardengine_test#TestPrune`
- `internal/boardengine_test#TestMergeTasks`
- `internal/boardengine_test#TestRunLock`
- `internal/boardengine_test#TestPromote`
- `internal/boardengine_test#TestUpsertFieldAllowlist`
- `internal/boardengine/doc.go`

**Uses:**
- `plan:internal/boardengine_test#newProjectBoard`

**Intent:**
D2: membership is `project` on a task or note and the plan order `order` on a project.
`UpsertPayloadKeys` admits `project` and `order`.
`Config.PairExists func(slug string) bool` reports whether an entry's pair exists on this machine; `Board` copies it onto every store it builds as `Store.pairExists`; nil reads every pair as absent.

`checkProjectRules` gains the membership rules, judged only for an entry whose `project` or `order` the write sets or changes from the loaded entries (an entry new to the board counts as set):
- a `project` naming an absent entry, a non-project entry, the entry itself, a done project, a project whose status is a run status (`IsRunStatus`), or a project chain with a cycle;
- an `order` naming an entry whose `project` is not this project, or naming one entry twice;
- a change of `project` on an existing entry whose pair exists (`pairExists`), naming `lyx batten cancel <slug> --remove` as the step before the change.
Each refusal names the entry, the field and the slug at fault, with the way forward of correcting that slug or dropping it from the field.
An entry whose `project` is unchanged is never refused, so marking a project done, by hand or by the landing's teardown settle, refuses nothing.
The inputs that reach them are upserts and batches carrying `project` or `order`, which `TestProjectRules` drives row by row through a `Board` built with a `PairExists` fake.

`Store.Rename` refuses a project whose `Beam` is true, saying a beam's slug stays fixed while it lives and naming `lyx board upsert` of its title as the way forward, and any entry whose pair exists, naming `lyx batten cancel <slug> --remove`; otherwise `renameSlug` rewrites every `project` and `order` naming the old slug as it rewrites `depends_on`.
`Store.Prune` drops the pruned slugs from every `order`, and keeps a done project while an entry the same prune does not remove names it in `project`.
`scopeChanged` strips the removed done slugs from `before.Order` as it does from `before.DependsOn`, so the run lock lets prune's `order` rewrite through on a held project.
`Store.MergeTasks` carries the removed entries' memberships to the upserted entry: an upserted entry with no `project` takes the first removed entry's, and every `order` naming a removed slug names the upserted slug once in its place; it refuses to remove a project another surviving entry still names in `project`, naming those entries as the ones to move first.
A promote keeps a note's `project` (no code change; a `TestPromote` row pins it).

Test rows: `TestProjectRules` reaches each refusal above and passes a done-mark of a project its members still name;
`TestRename_Refusals` refuses a beam project naming `lyx board upsert` and an entry with a pair naming `lyx batten cancel`;
`TestRename_KeepsEntryRewritesDependentsAndReportsMentions` rewrites `project` and `order`;
`TestPrune` drops a pruned slug from `order` and keeps a done project a surviving member names;
`TestRunLock` lets prune's `order` rewrite through on a run-held project and refuses a hand change of its `order`;
`TestMergeTasks` keeps a removed member's membership, rewrites a project's `order` naming a removed slug to name the upserted slug once, and refuses removing a still-named project naming its members;
`TestRename_Refusals`' rows with a pair and `TestProjectRules` build their boards with `newProjectBoard`, setting `PairExists` through its `configure` hook, since `newRenameBoard` sets no seam;
the package doc's Rename section, which says a rename rewrites only `depends_on`, is rewritten to name `project` and `order`;
`TestPromote` keeps `project`; `TestUpsertFieldAllowlist` accepts `project` and `order`.

**ImpactSummary:** Board write validation, rename, prune and merge gain the membership rules; New's and Config's callers gain one optional seam that nil keeps inert.
