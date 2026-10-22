# Card 4 — board-owner-writes

**Edit:**
- `internal/boardengine#Config`
- `internal/boardengine#Board`
- `internal/boardengine#New`
- `internal/boardengine#Board.boardCriticalSection`
- `internal/boardengine#Board.loadStore`
- `internal/boardengine#Store`
- `internal/boardengine#UpsertPayloadKeys`
- `internal/boardengine#Store.UpsertTask`
- `internal/boardengine#Store.UpsertTasksBatch`
- `internal/boardengine#Store.MergeTasks`
- `internal/boardcli#Command`
- `internal/boardcli#storeVerbs`
- `internal/boardengine_test#TestUpsertFieldAllowlist`
- `internal/boardengine_test#TestRunLock`
- `internal/boardengine/doc.go`
- `plugins/ly/skills/board/SKILL.md`

**Create:**
- `plan:internal/boardengine#checkOwnerPayload` -> `func checkOwnerPayload(s *Store, fields map[string]any) error`
- `plan:internal/boardengine#OwnerRequest` -> `type OwnerRequest struct`
- `plan:internal/boardengine#OwnerWrite` -> `type OwnerWrite struct`
- `plan:internal/boardengine#ErrOwnerOriginUnreachable` -> `var ErrOwnerOriginUnreachable = errors.New("board: origin is unreachable, so the owner write was not pushed")`
- `plan:internal/boardengine#SetOwners` -> `func SetOwners(boardDir string, req OwnerRequest) (OwnerWrite, error)`
- `plan:internal/boardengine#ReadOwners` -> `func ReadOwners(boardDir string) (Owners, error)`
- `plan:internal/boardengine#TestSetOwners` -> `func TestSetOwners(t *testing.T)`

**Uses:**
- `plan:internal/fabricengine#ParentSlug`
- `plan:internal/boardengine#ParseOwners`
- `plan:internal/boardengine#Owners.Effective`
- `internal/boardengine#ApplySkipEnv`
- `internal/fabricengine#Bolt.PullThenCommitWritten`
- `internal/fabricengine#Bolt.PushRecorded`

**Intent:**
D2's owner setter and the payload rule.
`Config` gains `ParentOf func(slug string) (string, error)`, copied onto `Board` by `New` and onto each `Store` that `boardCriticalSection` and `loadStore` build (`Store` gains `parentOf`), so reads and writes resolve parents alike; nil reads every parent as the prime.
`UpsertPayloadKeys` gains `owner_machine`.
`checkOwnerPayload` runs from `Store.UpsertTask`, `Store.UpsertTasksBatch` (per element) and `Store.MergeTasks` after `validateUpsertFields`: a payload carrying `owner_machine` equal to the entry's effective owner (`Owners.Effective` over the store's entries, prime owner and `parentOf`) is dropped from the fields as a no-op, and any other value refuses with "entry %q is owned by %q, and a board write never changes an owner: run `lyx adopt <slug>` from this machine's prime to take it over"; the replay applies through `changeEntry`, never these paths, so a replayed owner write is never refused.
`OwnerRequest` holds `Machine`, `Slugs`, `Prime`, `Takeover`, an optional `Claim` run status for one slug, `Sleep func(time.Duration)`, the retry's wait, nil reading `time.Sleep` (`PATTERN-test-speed`), and `BeforePush func()`, run once between the setter's commit and its first push and nil in production, the test's only way to race a second clone, since `PullThenCommitWritten` pulls immediately before the commit; `OwnerWrite` reports `Recorded` (the slugs, and `prime` when asked, whose owner reads as `Machine` after the push) and `Lost` (slug to the owner that stands instead).
`SetOwners` refuses an empty `Machine` naming `internal/machinename`'s error, then under `fabricengine.NewBolt(boardDir).PullThenCommitWritten` writes one commit "board: owner <machine> for <slugs>" setting each named task's `OwnerMachine`, the prime owner when `Prime` is set, and the claim status on its slug (an absent or done entry gets no claim and no owner, as today's claim skips it), loading and saving the store directly under the lock that call holds, then rendering.
Without `Takeover` a slug whose effective owner, read after the pull, is another machine is neither written nor claimed and is reported in `Lost` with that owner, so only `lyx adopt` moves an owner a start or resume finds set elsewhere.
A `fetch_failed` pull skip is `ErrOwnerOriginUnreachable` wrapped with its detail, and a `diverged` or `dirty` skip refuses naming its detail, which ends in its way forward.
It then pushes through `PushRecorded`; a transport failure is `ErrOwnerOriginUnreachable` wrapped, and any other failed push is returned, a classified failed push naming its class's way forward.
A push deferred by a held lock is retried once after `Sleep` waits one second, and a branch still ahead of its upstream afterwards refuses naming `lyx board sync` and then a re-run of the verb.
The skip settings are read through `ApplySkipEnv` as every board write reads them: under `BOARD_SKIP_GIT` the write lands in `board.json` with no pull, commit or push, and under `BOARD_SKIP_PUSH` it commits with no pull or push; either reads the written owners back as recorded.
After the push it re-reads the owners with `ReadOwners`, which parses the local `board.json` through `ParseOwners`, and reports each slug whose recorded owner is not `Machine` in `Lost`.
`boardcli`'s `Command` fills `ParentOf` over `fabricengine.ParentSlug` when its location is in a hub, and `storeVerbs`' upsert `Use` and help name `owner_machine` ("only the entry's effective owner, a no-op; `lyx adopt` changes an owner").
`doc.go` documents the owner field, the prime owner record, the effective owner, the setter as the sole writer and the payload rule, and the board skill's Claimed entries section says a claimed entry's owner changes only through `lyx adopt`.
`TestUpsertFieldAllowlist` gains rows: `owner_machine` equal to the effective owner is accepted and stores nothing new, another value is refused naming `lyx adopt`, and the same through merge's upsert and through an upsert-batch element.
`TestRunLock` gains a row: an accepted upsert that names a held entry's own effective `owner_machine` leaves board.json byte-identical.
`TestSetOwners` (integration, a board repo over a bare remote as `sync_integration_test.go` builds) covers: an empty machine refused writing nothing; a write recorded and pushed; the owner of a run-held entry changed with `Takeover`; a slug owned elsewhere skipped and reported in `Lost` without `Takeover`; a competing write losing to origin's, where `BeforePush` pushes a different `owner_machine` for the slug from a second clone, so the first push is rejected, the retry replays, origin's value stands, and the post-push re-read reports the slug in `Lost` with origin's owner; an origin URL nothing answers (`http://127.0.0.1:1/board.git`) reaching the pull's `fetch_failed` skip and returning `ErrOwnerOriginUnreachable` with nothing committed; a push-side transport failure, where `BeforePush` runs `git remote set-url origin http://127.0.0.1:1/board.git` in the board repo after the pull's fetch succeeded, so the first push fails on transport, which is not a retryable refusal and runs no backoff, and `SetOwners` returns `ErrOwnerOriginUnreachable` wrapped with the push's detail, with the owner commit kept local and the slug not in `Recorded`; a diverged pull, where a local commit of `notes/c.md` in the board repo meets a second clone's pushed commit of a different `notes/c.md`, so the pull's replay conflicts and the setter refuses naming `SkipDetail` and writes nothing (a `board.json` conflict cannot be the input, because the replay merges the board per entry and per field and never conflicts there); a dirty pull, where a tracked `notes/x.md` is left edited and uncommitted in the board repo while a second clone pushes a change to it, so the pull's fast-forward is blocked and the setter refuses naming the dirty detail, which names `lyx board sync`, and writes nothing; a deferred push, where `BeforePush` pushes a commit touching another entry from a second clone and takes the board write lock with `lock.TryAcquireWriteLock` on `fabricengine.BoardWriteLockFile`, so the first push is rejected and `retryBoltPush` defers because its try-lock fails, with a `Sleep` stub that releases the lock on its first call for the retried-and-lands case, which reports the slug in `Recorded` with its owner on origin, and keeps it for the still-ahead case, which refuses naming `lyx board sync`, so neither case waits a real second; the prime owner write; the claim status written in the same commit; `BOARD_SKIP_GIT` writing the owner with no commit.

**ImpactSummary:** boardengine write paths gain one payload rule and a new setter; boardcli wires the parent resolver.

**Commit:** 4: board: the owner setter and the owner payload rule
