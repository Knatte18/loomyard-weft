```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1 to F5 as the fixer carried them out.

**Findings**

1. **MEDIUM** — `internal/loomshed/raddlefile.go` `Call` (the `resumed` branch), proposal F1.
   - **Summary:** The resume path skips `verify`, including the interactive approval-hash match, for a proposal edited after the crash.
   - **Evidence:** `AlreadyFiled` compares only the slugs of the proposal now on disk and the body-prefix marker.
   - **Case:**
     - The board save lands for proposal P1 and the store write fails.
     - The proposal is edited into P2 with the same slugs, so the approval is stale or cleared.
     - The retry has `StoreDone` false and `AlreadyFiled` true, so `verify` and the approval check are skipped.
     - `stageAndFile` then stages P2 and `FileRaddle` takes its held branch.
     - The store records P2's `proposal.yaml` and hash while the board entries came from P1, and the unapproved edit is recorded as filed.
   - **Pinned by tests:** The `ADifferentHashOrCreatedStampIsNotTheDoneMarker` rows in `internal/boardengine/raddlefile_test.go` assert exactly this rewrite of the store under the new hash.
   - **Possible narrowing:** Skip only the board re-check on resume and keep the approval match, or have the held test also require the stored or staged hash to equal the filed one.

2. **LOW** — `internal/loomshed/raddlefile.go` `Call`, proposal F1.
   - **Summary:** The resume path drops the landed-entries list from the parent notice.
   - **Evidence:** `landed` stays nil when `verify` is skipped, so `raddleFiledNotice` omits "entries the proposal weighed that landed meanwhile".

3. **LOW** — `internal/loomshed/recordwrite_test.go`, test `RecordWithoutSupportLogGetsTheLog`, proposal F5.
   - **Summary:** The test's failure message is now false.
   - **Evidence:** It says "the board is not read while the record exists". With the log missing, `Call` now invokes `ReadStored` (a board read) even though the record exists. A board outage that failed a resume only when the record was missing now also fails it when only the log is missing.

F2, F4 and the F3 tests: no findings. `NoticeSession < Session` only affects the interactive approve path, and every seeded approve row was updated.

**Further tool calls:** none. My context already held every changed file.
