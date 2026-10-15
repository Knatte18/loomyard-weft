# Card 3 — settle-records-shedrun

**Create:**
- `plan:internal/shedrun#StopKey` -> `type StopKey struct`
- `plan:internal/shedrun#SettleRecord` -> `type SettleRecord struct`
- `plan:internal/shedrun#SettlerGoRule` -> `const SettlerGoRule = "go-rule"`
- `plan:internal/shedrun#SettlerTackler` -> `const SettlerTackler = "tackler"`
- `plan:internal/shedrun#SettlerArbiter` -> `const SettlerArbiter = "arbiter"`
- `plan:internal/shedrun#SettlerParent` -> `const SettlerParent = "parent"`
- `plan:internal/shedrun#SettleRestep` -> `const SettleRestep = "restep"`
- `plan:internal/shedrun#SettleVerifyRetry` -> `const SettleVerifyRetry = "verify-retry"`
- `plan:internal/shedrun#SettleHandOver` -> `const SettleHandOver = "hand-over"`
- `plan:internal/shedrun#SettleSpawn` -> `const SettleSpawn = "spawn"`
- `plan:internal/shedrun#SettleHandBack` -> `const SettleHandBack = "hand-back"`
- `plan:internal/shedrun#SettleBusyExit` -> `const SettleBusyExit = "busy-exit"`
- `plan:internal/shedrun#SettleRunningExit` -> `const SettleRunningExit = "running-exit"`
- `plan:internal/shedrun#SettleMoved` -> `const SettleMoved = "moved"`
- `plan:internal/shedrun#ParseSettleRecords` -> `func ParseSettleRecords(data []byte) ([]SettleRecord, []error)`
- `plan:internal/shedrun#TestParseSettleRecords` -> `func TestParseSettleRecords(t *testing.T)`
- `internal/shedrun/settle_test.go`

**Delete:**
- `internal/sheddriver#SettlerGoRule`
- `internal/sheddriver#SettlerTackler`
- `internal/sheddriver#SettlerArbiter`
- `internal/sheddriver#SettlerParent`
- `internal/sheddriver#SettleRestep`
- `internal/sheddriver#SettleVerifyRetry`
- `internal/sheddriver#SettleHandOver`
- `internal/sheddriver#SettleSpawn`
- `internal/sheddriver#SettleHandBack`
- `internal/sheddriver#SettleBusyExit`
- `internal/sheddriver#SettleRunningExit`
- `internal/sheddriver#SettleMoved`

**Edit:**
- `internal/sheddriver#StopKey`
- `internal/sheddriver#SettleRecord`
- `internal/sheddriver#ReadSettleRecords`
- `internal/sheddriver#GoVerifyRetrySpent`
- `internal/sheddriver#TestSettleRecords`
- `internal/sheddriver/drive.go`
- `internal/sheddriver/driverules.go`
- `internal/sheddriver#TestDecideDrive`
- `internal/sheddriver#TestGoDriver_Settle`
- `internal/sheddriver/doc.go`
- `internal/shedrun#RunTreeRels`
- `internal/shedrun#TreeRunRels`
- `internal/shedrun#TestRunsRootRel`
- `internal/shedrun/doc.go`
- `internal/arbiterstop/holdreading.go`
- `internal/arbiterstop/holdreading_test.go`
- `internal/battencli/drive.go`
- `internal/battencli/publishretry_test.go`
- `internal/loomcli/drivewiring.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Move the settle-record format into `internal/shedrun`, which already names the file and is on `reportengine`'s import allowlist, so the timing report (card 10) decodes settle records without importing `sheddriver`.

`internal/shedrun/settle.go` declares `StopKey` (`current_producer`, `history_length`, `state`, the JSON tags unchanged), `SettleRecord` (`stop_key`, `settler`, `action`, `outcome`, `at`, and the omit-empty `build` and `target`, unchanged), the four settler names and eight action names with their values unchanged, and `ParseSettleRecords`, the tolerant decoder over a file's bytes: one record per non-blank line in order, an unknown key read and dropped, and each line that does not decode skipped and returned as an error naming its 1-based line number.
It reads no file and imports nothing new: `shedrun` keeps its stdlib, `lyxcwd` and `lyxdirs` imports.
`RunTreeRels` gains `SettleRecords`, the tree-relative `go-settle.jsonl` path under the run directory, built from `settleRecordsFileName`, and `TreeRunRels` fills it; `TestRunsRootRel`'s expected `RunTreeRels` gains the field.
`TestParseSettleRecords` in `internal/shedrun/settle_test.go` covers two records in order, a blank line skipped, an unknown key decoding, and a torn line skipped with an error naming line 3 while the records before and after it still decode.

`sheddriver` keeps the file's append and read: `StopKey` and `SettleRecord` become aliases (`type StopKey = shedrun.StopKey`, `type SettleRecord = shedrun.SettleRecord`), so every `sheddriver` signature and `loomengine.ArbiterAttempt`'s field keep their spelling and no member is re-signed.
`ReadSettleRecords` reads the file and decodes it through `shedrun.ParseSettleRecords`, logging one Warn per returned error as it logs each skipped line today; an absent file still reads nil.
The twelve constants leave `sheddriver`, and every reference retargets to `shedrun`: `GoVerifyRetrySpent`, `TestSettleRecords`, `drive.go`, `driverules.go`, `TestDecideDrive` and `TestGoDriver_Settle` in `sheddriver`, and `internal/arbiterstop/holdreading.go`, `internal/arbiterstop/holdreading_test.go`, `internal/battencli/drive.go`, `internal/battencli/publishretry_test.go` and `internal/loomcli/drivewiring.go` outside it.
`TestSettleRecords` keeps only what `sheddriver` still owns: the append-and-read round trip with an absent file reading nil, a torn last line skipped by `ReadSettleRecords` while the record before it reads, and the verify-retry cap read; its unknown-key assertion goes, since `TestParseSettleRecords` pins the decoding.
`internal/shedrun/doc.go` says the settle-record format and its tolerant decoder are declared here and `sheddriver` appends and reads the file; `internal/sheddriver/doc.go`'s settle-record paragraph says the same from its side.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/shedrun` to 25.

**ImpactSummary:** the record's JSON is byte-identical; sheddriver's type names survive as aliases so only the twelve constant references in eleven files retarget, and RunTreeRels gains one field read by its one test.

**Commit:** 3: Move the settle-record format and its tolerant decoder into shedrun
