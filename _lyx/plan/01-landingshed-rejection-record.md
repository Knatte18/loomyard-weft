# Card 1 — landingshed-rejection-record

**Edit:**
- `internal/landingshed/approval.go`

**Create:**
- `internal/landingshed/rejection.go`
- `internal/landingshed/rejection_test.go`

**Uses:**
- `internal/landingshed#Approval`

**Intent:**
Add the operator-rejection record beside the approval record, in the package the PR gate will read both from (D4).
`rejection.go` declares `Rejection{PRNumber int "pr_number"; HeadSHA string "head_sha"; RejectedAt string "rejected_at"; Findings string "findings"}`, with `WriteRejection(path string, r Rejection) error` and `ReadRejection(path string) (Rejection, bool, error)` mirroring `WriteApproval`/`ReadApproval`:
an absent file is `found == false` with a nil error, and unreadable, undecodable, or field-missing content is an error (an empty or whitespace-only `Findings` counts as missing).
Also add `RemoveRecord(path string) error`, which deletes a decision record and treats an absent file as success;
`approve` and `reject` both call it to clear the other verb's pending record, and the rework row calls it once a round lands.
Move `WriteApproval`'s temp-file-then-rename body into one unexported atomic JSON writer that both `WriteApproval` and `WriteRejection` call, so the atomicity lives once.
Update `approval.go`'s file comment: the gate reads the record, not Publish (the reader moves in a later card, but the comment states the target shape once here).
`rejection_test.go` covers a round trip, an overwrite, an absent file, invalid JSON, each missing field including blank findings, and `RemoveRecord` on a present and an absent file.

**ImpactSummary:** Refactors WriteApproval's body onto a shared writer; its behaviour and its three callers are unchanged.
