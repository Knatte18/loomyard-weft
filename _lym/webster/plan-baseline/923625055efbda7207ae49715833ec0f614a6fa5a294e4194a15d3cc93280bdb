# Card 4 — usage-record-writer

**Create:**
- `plan:internal/lymdirs#UsageDirName` -> `const UsageDirName = "usage"`
- `plan:internal/shedrun#UsageDirRel` -> `func UsageDirRel() string`
- `plan:internal/shedrun#UsageDir` -> `func UsageDir(l *lymcwd.Location) string`
- `plan:internal/shedrun#ModelUsage` -> `type ModelUsage struct`
- `plan:internal/shedrun#SessionUsageRecord` -> `type SessionUsageRecord struct`
- `plan:internal/shedrun#RunUsageRecord` -> `type RunUsageRecord struct`
- `plan:internal/shedrun#LandedSizeRecord` -> `type LandedSizeRecord struct`
- `plan:internal/shedrun#RunUsage` -> `type RunUsage struct`
- `plan:internal/shedrun#WriteSessionUsage` -> `func WriteSessionUsage(dir string, rec SessionUsageRecord) error`
- `plan:internal/shedrun#WriteRunUsage` -> `func WriteRunUsage(dir string, rec RunUsageRecord) error`
- `plan:internal/shedrun#WriteLandedSize` -> `func WriteLandedSize(dir string, rec LandedSizeRecord) error`
- `plan:internal/shedrun#ParseRunUsage` -> `func ParseRunUsage(files map[string][]byte) (RunUsage, error)`
- `plan:internal/shedrun#TestUsageRecords` -> `func TestUsageRecords(t *testing.T)`
- `plan:internal/shedadapters#RecordSessionUsage` -> `func RecordSessionUsage(dir, row string, end shuttleengine.SessionEnd, reads shuttleengine.RecordReads)`
- `plan:internal/shedadapters#TestRecordSessionUsage` -> `func TestRecordSessionUsage(t *testing.T)`

**Edit:**
- `PATTERN.md`

**Extends:**
- `internal/lymdirs/doc.go`
- `internal/shedrun/doc.go`
- `internal/shedadapters/doc.go`

**Uses:**
- `plan:internal/shuttleengine#SessionEnd`
- `internal/shuttleengine#RecordReads`
- `internal/shuttleengine#TranscriptReader`
- `internal/shuttleengine#MessagePricer`
- `internal/shuttleengine#TranscriptEntry`

**Intent:** D12's one usage writer in shed's shared machinery (`PATTERN-shed-shared-machinery`) and the record files under a run's tracked `_lym/usage/`.
`lymdirs.UsageDirName` is the only place `usage` is named under `_lym` (`PATTERN-durable-vs-ephemeral-state`); `shedrun.UsageDirRel` joins it onto `lymdirs.LymDirName`, and `UsageDir` anchors it at the worktree's anchor path, like the other run paths.
Record files, all JSON written through `internal/state`'s atomic write and read tolerantly (`PATTERN-tolerant-records`: an unknown key is kept, never refused):
- `session-<session id>.json`, a `SessionUsageRecord`: `row`, `role`, `model`, `engine`, `input_tokens`, `cache_write_tokens`, `cache_read_tokens`, `output_tokens` (each `*int64`, null when unknown), `cost_usd` (`*float64`, null when unknown), `started`, `ended`, and `models`, a list of `ModelUsage` (model, the four counts, cost) present only when the session's messages, a fork's included, ran on more than one model.
  A later write for the same session id replaces the file, so a resumed session's later end overwrites it.
- `run.json`, a `RunUsageRecord`: `recipe`, `profile`, `stored_record` (bool); written by card 6's Seed.
- `landed.json`, a `LandedSizeRecord`: `lines_added`, `lines_deleted`, `files_changed`; written by card 6's Finalize.
`ParseRunUsage` is the one reader of the directory's files, given as a name-to-bytes map the way a git tree read hands them (the report reads them that way), returning `RunUsage` (the run record, the landed record, each with a found flag, and the session records); a file that does not decode is an error naming it.
`shedadapters.RecordSessionUsage` is the one writer of a session record and never returns an error or stops anything:
- With a `TranscriptReader` in `reads`, it locates the session among `reads.Transcripts.LocateSessions(end.Worktree)` by `end.SessionID`, reads it with `ReadSession`, and for every entry of the main transcript and of each fork carrying `Usage` adds its four counts to its model's total and prices it at its own model through `reads.Pricer.PriceMessage`, as `tools/tokencount`'s `messageCost` does; a message whose model has no price, or no pricer at all, leaves the cost unknown.
  `model` is the model of the main transcript's messages (the one most of its priced messages ran on).
- With no `TranscriptReader`, such as Codex, or when locating or reading any transcript, a fork's included, fails, it takes the four counts from `end.Usage` (each unknown when `SplitKnown` is false) with the cost unknown; a locate or read failure is logged at Warn.
- A failed write of the file is logged at Warn and stops nothing.
`TestUsageRecords` (tier0) round-trips each record through its writer and `ParseRunUsage`, keeps an unknown key, overwrites a session file of the same id, and names a file that does not decode.
`TestRecordSessionUsage` (tier0, a fake `TranscriptReader` and fake `MessagePricer` in the test file) has rows: one model priced per message with the counts summed; a fork on another model priced at the fork's model with `models` holding both; an unpriced model leaving `cost_usd` null; no transcript reader (counts from `SessionUsage`, cost null); a locate failure and a read failure of a fork (counts from `SessionUsage`, cost null, a Warn logged, a file still written); and an unwritable directory (no panic, no error).
`PATTERN.md` gains the declared entry in the `## Shed and loom` group, after `PATTERN-shed-shared-machinery`:

```
- `PATTERN-run-usage-record` — Recording an agent session's tokens and cost per session under a run's `_lym/usage`: only shed's one usage writer, one file per session; a cost report reads token counts and session costs only from those files, never from a review round's usage record.
```

The new entry grows the overview past the headroom line, so this card also prescribes the parent's eight recorded `changes` decisions (Added after Discussion, 2026-11-05), each entry's whole overview line replacing its current one, its `(test)` marker and background link kept as the record's text and the current line give them:

```
- `PATTERN-shed-recipe-registry` — Adding producer, recipe row or block gate: one `Constructor` or `BlockGates` map in `shedrecipe`, bar `Implementer`'s; row values by key, side actions as `Command` rows, off switches per the background. — [background](pattern/PATTERN-shed-recipe-registry.md)
- `PATTERN-test-speed` — Writing or running a test: via `lym gate test` by gate key; one `tiergate.Require` tier, lowest that fits; it pins behavior no test covers; skips, clocks, `t.Parallel` and `//lym:seam` per the background. (test) — [background](pattern/PATTERN-test-speed.md)
- `PATTERN-stencil-ownership` — Reading prompt or spec: at call time via `stencilstore`, never embedded, bar the orch role, fixed in `contracts/orchrole` with no board copy; an instruction file points at another producer's format contract, never restating it. — [background](pattern/PATTERN-stencil-ownership.md)
- `PATTERN-cwd-resolution` — Resolving a cwd, worktree root or anchor: only `lymcwd`, from the worktree's weft-root `.lym-anchor`; only `fabricengine` writes or git-reads it; engines get absolute paths, never import lymcwd; hub containers and workspace files stay hub-reachable only. (test) — [background](pattern/PATTERN-cwd-resolution.md)
- `PATTERN-durable-vs-ephemeral-state` — Adding file: tracked under `_lym`, never-tracked under `.lym`, same subpath, bar heddle's rendered files and `.heddle` at a warp worktree's anchor; only `lymdirs` names either, bar a `tools/` script's variable block. — [background](pattern/PATTERN-durable-vs-ephemeral-state.md)
- `PATTERN-remote-refs-lock` — Writing remote-tracking refs by fetch, pull or push: under the repository's remote-refs lock via `gitrepo`, waiting while held; the pause gate, writes that take none and judges that hold it per the background. (test) — [background](pattern/PATTERN-remote-refs-lock.md)
- `PATTERN-slug-remote-order` — Touching a slug's pair or origin branches: under its teardown lock, shared for a pair push outside a removal, else exclusive, held by a teardown's detached child to its last remote step; an origin weft delete leases the tip read.
- `PATTERN-config-layers` — Loading config: only `configengine` reads, stacks, locates and judges layers against the template; callers use `Load` or `LoadOrTemplate`, bar `runprofile`; only `hubreconcile.Ensure` reconciles on a binary change; only `lym config reconcile` prunes unknown base-layer keys. — [background](pattern/PATTERN-config-layers.md)
```

Every rule the shortened lines drop is already stated in its entry's background file, so no rule text moves and no background file changes:
- `pattern/PATTERN-shed-recipe-registry.md` states the lookup by key (line 8), the `Command` row (line 22), the `entry_gate` off (line 27) and the off that reads more than one key decided in its own call (line 29);
- `pattern/PATTERN-test-speed.md` states the `//lym:seam` rule (lines 94-102), injected clocks (lines 104-108), `t.Parallel` by default (lines 110-113) and that a skip never names the OS alone (line 119);
- `pattern/PATTERN-remote-refs-lock.md` states the pause gate (line 5), the writes that take no lock (line 10) and the judge rule (line 11).
`PATTERN-slug-remote-order` has no background file; its shortened line holds, since only a pair push holds the teardown lock shared (`internal/fabricengine/teardownlock.go:81`, `internal/fabricengine/pushanchored.go:164`) and every other taker holds it exclusive (`teardownlock.go:114`, `remoteteardown.go:316` and `364`).
The three `doc.go` files each gain a paragraph: lymdirs names `UsageDirName`; shedrun names the usage directory and its three record files; shedadapters names `RecordSessionUsage` as the one writer.

**ImpactSummary:** New files and one new leaf constant; nothing calls the writer until card 5 wires it, and the PATTERN overview gains one entry while eight others shrink, every dropped rule already stated in its background file.
