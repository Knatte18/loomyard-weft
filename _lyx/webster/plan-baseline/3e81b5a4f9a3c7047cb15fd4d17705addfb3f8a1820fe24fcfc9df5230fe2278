# Card 8 — approve-review-file-refusal

**Edit:**
- `internal/parentreview#ErrEmptyReviewFile`
- `internal/parentreview#Store.RecordVerdict`
- `internal/parentreview#TestRecordVerdict`
- `internal/loomcli#reviewStoreErr`
- `internal/loomcli#TestReviewVerbs_ReviewFileMissingOrEmpty`

**Prosa:**
- `contracts/specs/refusal-spec.md`

**Intent:**
D6: `lyx loom review approve --review` refuses a missing, unreadable or whitespace-only file exactly as `reject` does, and records no verdict; an approve with no `--review` is unchanged.

`Store.RecordVerdict` reads a given review path once for either kind and returns `ErrEmptyReviewFile` when the read fails or the content is whitespace-only;
with no path, a reject still returns `ErrEmptyReviewFile` and an approve records a verdict with no review copied.
The approve-only wrapped read error goes, so nothing returns a bare `fs.ErrNotExist` from the store any more.
`ErrEmptyReviewFile`'s doc and `RecordVerdict`'s doc say the rule covers a reject and an approve with a review file.

`reviewStoreErr` drops its `fs.ErrNotExist` clause, which no store error reaches now, and `review.go` drops the `io/fs` import with it; the empty-file message keeps naming the refused verb.

Tests:
`TestRecordVerdict` gains approve cases with a missing file, a whitespace-only file and an unreadable path, each returning `ErrEmptyReviewFile` and leaving `verdict.json` and `review.md` absent.
`TestReviewVerbs_ReviewFileMissingOrEmpty` gains an "approve empty" row with a whitespace-only file beside the existing "approve missing" row, each refused with `review file is missing or empty` and a way forward naming `lyx loom review approve`.

`refusal-spec.md`'s `review: review file missing or empty` row says the file is missing, unreadable or whitespace-only, and its way forward is to write the review to a non-empty file and re-run the refused verb, `lyx loom review reject` or `lyx loom review approve --review`, with its path.

**ImpactSummary:** one store path narrows an approve with a bad review file to the reject's refusal; the CLI drops one now-unreachable error clause; approve without --review is unchanged.
