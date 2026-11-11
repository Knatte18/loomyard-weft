---
round: 1
focus:
  - "internal/boardengine/recordupsert.go: does the refusal order and set match D4 (new, record-less, done, run-held, worktree-holding, live weft, failed origin probe), each with a way forward and nothing written?"
  - "internal/boardengine/recordupsert.go: is the family-project exemption from the worktree and live-weft refusals gated on the raddle store's filing-done record, and does it still apply the run lock?"
  - "internal/boardengine/recordupsert.go: is the record file written before board.json under the board lock, and does a failed save or a refused payload restore the former record file as FileRaddle does?"
  - "internal/boardengine/decision.go, raddlefile.go, board.go: does every stored-record write (StoreDecisionRecord, the upsert, Raddle-File's filing) stamp record_body_sha256 with the sha256 of the body as that write leaves it?"
  - "internal/boardengine promote path: does the warning compare the body as loaded before promote's write against record_body_sha256, skip entries with no stamp, name `lym board upsert --record-file`, and never refuse?"
  - "internal/fabricengine/add.go and remoteleftover.go: do Add and the exported weft-liveness function share one probe under Add's own SkipGit and SkipPush conditions, with the local branch check alone deciding where Add makes no origin probe, and does an origin probe failure surface as an error rather than as not live?"
  - "internal/boardcli/cli.go and recordfile.go: is the run-agent refusal built on the board's existing strand-name classification, does the flag mirror --body-file, and does boardcli run no git?"
  - "internal/discussionparser/validate.go: does the record-only section check run without a support log, and does Validate still apply the same check to the record it reads with no behavior change for discussions?"
  - "Test economy: TestUpsertTaskWithRecord_FamilyProjectSkipsWorktreeAndWeftRefusals is listed as contained; compare its assertions with TestFileRaddle, TestUpsertTaskWithRecord_Refusals and TestUpsertTaskWithRecord_ReplacesRecordWithPayload before filing anything."
  - "Acceptance criteria coverage: is each D4 refusal, the family-project replacement, the refused-payload-keeps-record case and both D5 no-warning cases reached by a test?"
  - "Doc comments in doc.go files of the four packages: do they carry the new verb, stamp and function, and is the change description's claim in _lym/landing/summary.md borne out by the diff?"
---

The record's decisions D4 and D5 are the measuring stick; the review range is the three feature commits plus their test and fix commits.
