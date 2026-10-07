---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/fabricengine/add.go:75
    summary: Added comments break inside a clause, leaving a fragment on the next line (add.go list after a colon; remoteleftover.go compound predicate)
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/fabricengine/remoteleftover.go:70
    summary: Added non-test comment lines hold two or more independent clauses unbroken (remoteleftover.go, configchanges.go, doc.go)
  - id: F3
    severity: BLOCKING
    class: consistency
    location: internal/fabricengine/add_remoteleftover_integration_test.go:3
    summary: Added test comments hold independent clauses joined by a comma plus and or so on one line, with no break between them
---

Range: `product.parent` is `main` and no `_lyx/loom/rework/` round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD`.
`go build ./...`, `go vet -tags integration` and `go test` over the touched packages pass; the findings are all `PATTERN-comment-line-breaks`, which the repo makes blocking.
`PATTERN-comment-line-breaks` governs added and changed lines only.
Moved legacy lines are left alone: the `CommitWeftPaths records the` wrap now at `add.go:301-303` is the old comment re-indented into an `if`, not new text.
The `ErrRemoteLeftover` doc at `remoteleftover.go:31` keeps its list on the colon's line, so the third focus question is settled as not a defect.
The comments in `checkout.go`, `mutation.go`, `reconcile.go` and `landingshed/finalize.go` break correctly, so the first focus question is settled as not a defect for them; the breaks inside a clause are in the files named below.

### [BLOCKING] Breaks inside a clause

**Location:** `internal/fabricengine/add.go:75-76`, `internal/fabricengine/remoteleftover.go:2-3`
**Issue:** `add.go:75` ends on a comma inside a list that follows a colon, and `add.go:76` starts `and the warp branch from origin when it is there, ...`, a fragment with no verb of its own.
`remoteleftover.go:2` ends `either makes the pair live, so Add adopts it,` and `:3` starts `or is a removed pair's leftover ...`, a break between the two predicates of one compound predicate.
**Fix:** Join the `add.go` list onto one line, or move the colon's list wholly to the next line, so the break falls only at a sentence end or between independent clauses.
In `remoteleftover.go`, put both predicates on one line, then break before `so the refusal never arrives mid-Add as a rejected push.`.

### [BLOCKING] Independent clauses joined on one line

**Location:** `internal/fabricengine/remoteleftover.go:70`, `internal/fabricengine/remoteleftover.go:71`, `internal/fabricengine/configchanges.go:28`, `internal/fabricengine/doc.go:496`
**Issue:** Each line joins independent clauses with no break.
`remoteleftover.go:70` holds three (`an origin tip that equals ... needs nothing, a local tip ... becomes fastForwardTo, and a true divergence is refused`), and `:71` holds three (`an absent remote branch returns the zero answer, an origin tip ... is a replaceable leftover (...), and any other origin tip makes the pair live`).
`configchanges.go:28` is `The diff runs from the merge-base to the task tip, so a commit on the parent's side after the fork never appears.`
`doc.go:496` joins `a local weft branch behind origin is fast-forwarded ...` and `a weft branch only on origin becomes ...` with a comma.
**Fix:** Break after each independent clause: before `so a commit ...` in `configchanges.go:28`, before each clause of the two `remoteleftover.go` lines, and after the first clause in `doc.go:496`.

### [BLOCKING] Independent clauses joined in added test comments

**Location:** `internal/fabricengine/add_remoteleftover_integration_test.go:3`, `:88`, `:281`; `internal/fabricengine/add_rollback_adopt_test.go:118`; `internal/fabricengine/checkout_rollback_test.go:140`; `internal/fabricengine/origin_integration_test.go:325`; `internal/fabriccli/adoptremote_integration_test.go:4`, `:23`, `:70`, `:302`, `:369`; `internal/landingshed/finalize_integration_test.go:139`, `:143`; `internal/loomcli/landingdeps_integration_test.go:3`, `:4`, `:26`
**Issue:** Each added line holds two or more independent clauses joined by a comma plus `and` or `so`, or by commas, with no break.
Two examples: `add_remoteleftover_integration_test.go:88` ends `the pair is live, so Add adopts it.`, and `landingdeps_integration_test.go:4` ends `... for the whole binary, so this file adds no TestMain.`
**Fix:** Break each line before the second clause's conjunction (`, so ...`, `, and the ...`, `, origin's copy ...`).
Compound-predicate and list sentences among the added test comments stay as they are.

## Focus departures

None.
