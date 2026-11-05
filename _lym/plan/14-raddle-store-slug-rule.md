# Card 14 — raddle-store-slug-rule

**Create:**
- `plan:internal/boardengine#RaddleStoreEntry` -> `type RaddleStoreEntry struct`
- `plan:internal/boardengine#RaddleStore` -> `type RaddleStore struct`
- `plan:internal/boardengine#ReadRaddleStore` -> `func ReadRaddleStore(boardDir string) (RaddleStore, error)`
- `plan:internal/boardengine#RaddleStore.Holds` -> `func (s RaddleStore) Holds(slug string) (RaddleStoreEntry, bool)`
- `plan:internal/boardengine#RaddleStoreDir` -> `func RaddleStoreDir(boardDir, sketch string) string`
- `plan:internal/boardengine#TestReadRaddleStore` -> `func TestReadRaddleStore(t *testing.T)`
- `plan:internal/boardengine_test#writeRaddleStore` -> `func writeRaddleStore(t *testing.T, boardDir, sketch string, entry boardengine.RaddleStoreEntry)`

**Edit:**
- `internal/boardengine#checkNewSlugs` -> `func checkNewSlugs(loaded, after []Task, renamedTo string, validate func(slug string) error, store RaddleStore, storeErr error) error`
- `internal/boardengine#newSlugError` -> `func newSlugError(slug string, reason error, after []Task, validate func(slug string) error, store RaddleStore, promotion bool) error`
- `internal/boardengine#Board.boardCriticalSection`
- `internal/boardengine_test#TestSlugRule`
- `internal/boardengine/doc.go`

**Intent:** D10's raddle store on the Bolt and the board's refusal of a slug it holds.
The store is one directory per Sketch beside `board.json`, `raddle/<sketch slug>/` (`RaddleStoreDir`, the directory name declared once in boardengine), holding `proposal.yaml` (the approved proposal), `cold-proposal.yaml`, `decision-record.md` (the Sketch's record) and `store.json` (`created`, the Sketch's `created` stamp; `hash`, the approved proposal's hash; `filed`, every filed slug, the project's included; and, for a project or beam shape, `project`, which is also in `filed`).
Prune never touches it: `Store.Prune` and `followDecisionRecords` read only `board.json` entries and `decision-` files.
`ReadRaddleStore` reads every `store.json` under the store root into `RaddleStore`, an absent root holding none; `Holds` answers the entry of the Sketch slug, or of the Sketch whose `filed` holds the slug.
`boardCriticalSection` reads the store once under the lock, before `fn` mutates, so the rule judges the store as it stood before the write, and passes it and its read error to `checkNewSlugs`, which runs after `checkDiscussedPromotion` (card 13).
`checkNewSlugs` keeps its judged writes (an added task or project, a note made a task or project, a rename) and also refuses a made slug the store holds, as a Sketch slug or as a filed slug, whether or not the pair slug seam is set.
It exempts the made entry whose slug is a store entry's Sketch slug and whose `created` equals that entry's recorded `created`, so a live raddle run's own Sketch is never refused.
A store read error refuses only a write that makes a slug, naming the unreadable `store.json` and, as the way forward, restoring it from the Bolt's history with `git -C <bolt> checkout -- <path>`; a write that makes no slug goes on, so one broken file does not stop the board.
`newSlugError` names a slug both the seam and the store admit; for a note made a task or project, which keeps its slug, it names instead `lym board rename` of the note to such an admitted slug before the promotion, a note's rename not being judged.
The input that reaches each refusal: `lym board upsert` creating a task or project under a filed slug, `lym board rename` to a Sketch slug the store holds, and `lym board promote --recipe raddle` of a note under a filed slug (a filed part a motif run's end made a note again, which is discussed, so card 13's rule admits the raddle promotion and this rule refuses it).
Bound: the board's Bolt replayer adds an entry through `validateWrite` alone, never this rule, so a board commit from a machine that had not yet pulled the store can land an entry under a held slug on origin; batten's start guard (card 24) is the check that meets such an entry at its run.
`writeRaddleStore` writes a store directory's `store.json` into a test board's directory, for the rows below, before card 16's filing write exists.
`TestSlugRule` gains rows: a new task under a store-held Sketch slug and under a filed slug (refused, an admitted slug named); a new project under a filed project slug after that project was pruned (refused); a rename to a held slug (refused); a promotion with recipe `raddle` of a discussed note under a filed slug (refused naming `lym board rename` to an admitted slug); the entry whose `created` the store records for its own Sketch slug (admitted); a board built with a nil `ValidateSlug` and a store hit (still refused); an unreadable `store.json` refusing a slug-making write and letting a status write through.
`TestReadRaddleStore` (tier0) reads a fixture store of two Sketches, one of the project shape whose project slug `Holds` answers, an absent root and an unreadable `store.json` (error naming it).
`doc.go` gains a `# Raddle store` section.

**ImpactSummary:** One judged write rule widened by a store read under the lock; `checkNewSlugs`' one caller and its one test move with the re-sign, and every write that makes no slug is unaffected.
