---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    location: internal/shedengine/run.go:558
    summary: recordedVerdict doc comment leaves the independent clause after ', or it kept its length' on the same line
  - id: F2
    severity: LOW
    location: internal/battenshed/innerrun.go:83
    summary: approvalActedContent doc comment leaves the independent clause after ', so identity comparison alone decides' on the same line
  - id: F3
    severity: LOW
    location: docs/overview.md:430
    summary: changed batten bullet line leaves the independent clause after ', and every self-bounce repeats' unbroken
  - id: F4
    severity: LOW
    location: docs/overview.md:434
    summary: changed batten bullet line leaves the independent clauses after ', so their own reason is' and ', so a step-driven lifecycle reports' unbroken
  - id: F5
    severity: LOW
    location: internal/shedengine/status.go:70
    summary: new LastAt field comment leaves the independent clause after the semicolon (At keeps the first call's time) on the same line
  - id: F6
    severity: NIT
    location: internal/shedengine/run.go:503
    summary: changed persist doc line still carries the semicolon-joined independent clause 'write; it selects' on one line
---

Review range: `product.parent` is `main` (from `_lyx/shed/awaiting-status-history-growth/status.json`), merge base `724b78b50`, reviewed `git diff 724b78b50..HEAD` (cards 1–4 plus the round-1 fix commits F1–F7).

Substrate: `go build ./...`, `go vet ./internal/shedengine/... ./internal/battenshed/... ./internal/battencli/...`, and both `go test` lines of the plan's `verify:` block (plain and `-tags integration`) all pass on HEAD.

The code changes themselves (`appendOrFold`/`foldsInto`, `recordedVerdict`, the two-line approval-acted marker and its parser, the awaiting-branch split) carry no correctness finding this round.
Every finding below is the CONSTRAINTS.md Comment Line-Break Convention (Go) or the CLAUDE.md markdown semantic-line-break rule, applied only to new or changed lines:
a comma before a coordinating conjunction (and/or/so) or a semicolon, where what follows has its own subject and verb, takes a line break.

### [LOW] recordedVerdict doc comment: unbroken independent clause

**Location:** internal/shedengine/run.go:558
**Issue:** `the history got longer, or it kept its length while the last entry's Repeats changed, which is a fold.` — `it kept its length` has its own subject and verb after `, or`, so the convention requires a break there.
**Fix:** Break after `the history got longer,` and start the next comment line with `or it kept its length ...`.

### [LOW] approvalActedContent doc comment: unbroken independent clause

**Location:** internal/battenshed/innerrun.go:83
**Issue:** `The length rides beside the identity, never inside it, so identity comparison alone decides whether an approval is new.` — `identity comparison alone decides` is an independent clause after `, so`.
**Fix:** Break after `never inside it,` and start the next comment line with `so identity comparison alone decides ...`.

### [LOW] docs/overview.md batten bullet line 430: unbroken independent clause

**Location:** docs/overview.md:430
**Issue:** The changed `Those per-poll history writes` line keeps `the status commit skips a transition it has already committed, and every self-bounce repeats the same ... pair` on one line; `every self-bounce repeats` has its own subject and verb after `, and`.
**Fix:** Break after `already committed,` so `and every self-bounce repeats ...` starts its own continuation line (indented to the bullet's continuation column); leave unchanged neighbouring lines alone.

### [LOW] docs/overview.md batten bullet line 434: two unbroken independent clauses

**Location:** docs/overview.md:434
**Issue:** The split `status` line (new in this diff) keeps `(the other rows carry no on_stuck, so their own reason is the persisted error)` and `recorded by the producer rather than only returned, so a step-driven lifecycle reports it ...` on one line; both clauses after `, so` have their own subject and verb.
**Fix:** Break after `carry no `on_stuck`,` and after `rather than only returned,`, each continuation line starting with `so`.

### [LOW] LastAt field comment: unbroken semicolon clause

**Location:** internal/shedengine/status.go:70
**Issue:** `// LastAt is the RFC 3339 UTC time of the most recent folded call, empty when none; At keeps the first call's time.` — the new comment joins an independent clause after a semicolon on the same line.
**Fix:** Break after `empty when none;` and put `At keeps the first call's time.` on the next comment line.

### [NIT] persist doc comment: changed line still joins a semicolon clause

**Location:** internal/shedengine/run.go:503
**Issue:** The diff rewrote this line (`write; it selects ... wording of activity.last.`), and it still carries the independent clause `it selects ...` after the semicolon on the same line as the tail `write;` of the legacy-wrapped sentence above.
**Fix:** Move `write;` to the end of line 502 (the sentence is already a changed comment) and start line 503 with `it selects ...`.
