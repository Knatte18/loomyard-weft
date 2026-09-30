---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: contracts/specs/refusal-spec.md:40
    summary: The refusal table is missing rows for refusals the sweep cards named, so the new invariant's every-refusal-has-a-row clause fails on landing.
  - id: F2
    severity: MEDIUM
    location: contracts/specs/refusal-spec.md:60
    summary: Several Way forward cells disagree with the messages the code emits after the fixup commit reworded them, although the spec claims the cell quotes the message.
  - id: F3
    severity: MEDIUM
    location: internal/webstercli/recoverbatch.go:152
    summary: Two recover-batch refusals name lyx webster recover-batch NN-slug as the way forward, an argument the verb's own Atoi parse rejects.
  - id: F4
    severity: MEDIUM
    location: internal/webstercli/rebaseline.go:81
    summary: The new rebaseline verb's fabric-sync refusal carries no way forward, unlike every other webster fabric-sync refusal.
  - id: F5
    severity: MEDIUM
    location: contracts/stencils/webster/webster-template-master.md:108
    summary: Master's failure ladder has no rung for the batch_failed envelope recover-batch now emits, so a failed recovery falls through to generic error handling.
  - id: F6
    severity: LOW
    location: internal/shedengine/goto.go:52
    summary: Three goto refusals (missing status file, unknown target, missing --to) end without the trailing way forward clause the shared decision requires.
  - id: F7
    severity: LOW
    location: internal/websterengine/beginbatch.go:250
    summary: The two reworded fingerprint-mismatch refusals name rebaseline but carry no way forward marker, so they do not follow the way-forward-phrasing decision.
  - id: F8
    severity: LOW
    location: internal/websterengine/doc.go:142
    summary: The new websterengine package documentation names a nonexistent RerunCardVerifies, a fork-write correctness class that does not exist, and a record-batch fingerprint refusal that does not exist.
  - id: F9
    severity: LOW
    location: internal/shedengine/doc.go:29
    summary: Docs and comments the goto change falsified are left stale, including the fixed budget reason, the since-last-done budget rule, and several four-generic-verbs comments.
  - id: F10
    severity: LOW
    location: internal/websterengine/rebaseline.go:87
    summary: With no recorded StartSHA the rebaseline refusal reads reset the branch to the run's start commit  with git, naming no commit.
  - id: F11
    severity: LOW
    location: internal/shedengine/goto.go:2
    summary: Many new and changed Go comments are hard-wrapped mid-sentence, against CONSTRAINTS.md's Comment Line-Break Convention.
  - id: F12
    severity: LOW
    location: internal/webstercli/verbs_test.go:1184
    summary: The fabric-sync way-forward tests take the way forward by setting openFabric to nil, which makes fabricSync a no-op, so they never show that the saved state gets committed.
  - id: F13
    severity: NIT
    location: internal/battencli/cli.go:187
    summary: The batten parent Long's goto example uses --to Teardown, which names no batten row; the row is Worktree-Teardown.
  - id: F14
    severity: NIT
    location: internal/websterengine/batchfail.go:103
    summary: batchfail.go adds a containsString helper that duplicates slices.Contains, and the new engine rebaseline.go lacks the file header comment every sibling file carries.
---

The build, `go vet ./...`, `go test ./...`, `go test -tags integration` over every touched package and `go vet -tags smoke ./...` all pass.
The review range is `git diff $(git merge-base main HEAD)..HEAD`, with `main` read from `_lyx/shed/refusal-audit/status.json`'s `product.parent`.

Focus verdicts in brief:
RecordBatch follows card 8's order.
A `ClassifyViolation` error returns before any mutation, dispositioned identities are dropped before classification, the verify re-run also fires on `bs.AuditWarnings`, and `failBatch` consumes new transcripts idempotently, so a retry neither re-reports nor double-warns.
Parent ordinals index transcript-ordered, cumulative slices, so they stay stable as the session grows.
`canonicalPath` resolves the nearest existing ancestor, and `ignoredPath` reads a missing path as not ignored (the conservative correctness answer).
Every history reader in the tree handles a `goto` entry, and the budget reset stays within the target's segment.
`ReasonBounceBudgetExhausted` is matched only by prefix, in `loomengine/anomaly.go`, `loomrecipe`'s test and the shedengine tests, and ly-drive does not read the text.
The CLI saves and commits before it emits `batch_failed` or `report_archived`.
Rebaseline keeps batch records, refuses renumbered or regrouped begun batches, and reads a legacy record without `Cards` as `NN-<Slug>`.
`failBatch` reuses `archiveStaleReport` and does not duplicate it.
The open items are below.

### [MEDIUM] The refusal table lacks rows the sweep cards named

**Location:** `contracts/specs/refusal-spec.md` (webster, shed and loom sections)
**Issue:** Card 28 requires one row for every refusal that cards 5–27 named.
CONSTRAINTS.md's new Refusal Way-Forward Invariant says every reachable refusal "has a row in its module's section".
These refusals are reachable and were named by a card, but have no row:
webster: record-batch's mid-merge refusal, the head-SHA mismatch and non-merge-movement refusals, begin-batch's report-exists refusal, recover-batch's OK-report refusal, record-batch's recovery-batch (`not a fork batch`) refusal, the run-exit refusals (done with missing or malformed summary, batches lacking a done record, nil fork audit, audited-count shortfall, missing integration report), `no run in progress`, run's `--plan-dir` override refusal, recover-batch's vanished-state refusal, and the Webster row's `paused out of band` stop;
shed: the missing-seed refusal (`shedrun.MissingSeedMessage`), status's `--watch --json` refusal, and the recipe-told absent-status refusal;
loom: approve's missing status file and start's not-parked refusal.
**Fix:** Add one row per refusal, each Way forward cell quoting what its message names.

### [MEDIUM] Way forward cells disagree with the reworded messages

**Location:** `contracts/specs/refusal-spec.md:53`, `:73`, `:112`, `:47`, `:74`
**Issue:** The spec says the Way forward cell quotes the message's clause "so the cell and the message cannot disagree".
Fixup commit b6675de6f reworded the run-exit stuck reason to "in the task worktree with git" and the preflight clauses to "the code changes"/"_lyx", but the cells still say "on the warp" and "commit or stash the warp with git and `lyx fabric commit` the weft".
The fingerprint-mismatch row's trigger names record-batch, but record-batch runs no fingerprint check; only begin-batch and run do.
The re-baseline-persist row offers `lyx webster rebaseline` for the bracket verbs, but their message says only "re-run the same verb".
**Fix:** Rewrite those cells to quote the emitted clauses, and drop record-batch from the fingerprint trigger.

### [MEDIUM] recover-batch names an argument its own parse rejects

**Location:** `internal/webstercli/recoverbatch.go:152`, `:228`
**Issue:** `batchName` is `NN-<slug>`, so the reed-boot and vanished-state refusals say "re-run `lyx webster recover-batch 03-foo`".
The verb parses its argument with `strconv.Atoi`, which refuses `03-foo` as "not a valid batch number", so the named way forward is itself a refusal.
The reaching test checks only for the substring `recover-batch` and then re-runs with `1`, so it cannot catch this.
**Fix:** Name the batch number (`%02d` of `batchNumber`) in both clauses, and have the tests assert the exact runnable command.

### [MEDIUM] rebaseline's fabric-sync refusal has no way forward

**Location:** `internal/webstercli/rebaseline.go:81`
**Issue:** `webster: plan rebaselined but the fabric sync failed: %v` ends there.
This verb was added in the same diff that gave every other fabric-sync refusal `fabricSyncWayForward`, and the invariant requires a way forward.
**Fix:** Append `fabricSyncWayForward`, as the bracket verbs do.

### [MEDIUM] Master has no rung for recover-batch's batch_failed

**Location:** `contracts/stencils/webster/webster-template-master.md:108`
**Issue:** Card 12 makes recover-batch emit `{"batch_failed": true}` when the recovery's post-batch checks reject it.
The ladder covers `batch_failed` only from record-batch ("call recover-batch, then follow the recover-batch rungs").
The recover-batch rungs name `running`, terminal `done` and terminal `stuck`/`dead`, but no `batch_failed`.
A Master whose recovery fails its done-checks gets no instruction, and read literally the record-batch rung suggests another recover-batch, an unbounded loop.
**Fix:** Add a recover-batch `batch_failed` rung that treats it as a failed recovery: write `outcome: stuck` naming the batch and stop, as for terminal `stuck`/`dead`.

### [LOW] goto refusals without the trailing way forward

**Location:** `internal/shedengine/goto.go:52`, `:65`; `internal/shedverbs/goto.go:35`
**Issue:** The missing-status refusal copies step's pre-sweep text ("Shed never seeds one"), although step's own now ends with a way forward.
The unknown-target and missing-`--to` refusals list the valid names but carry no `way forward:` clause.
Card 18 is under the way-forward-phrasing decision.
**Fix:** End each with a `way forward:` clause: the step seed text for the missing file, and "re-run goto with --to set to one of: …" for the two target refusals.

### [LOW] Fingerprint-mismatch refusals lack the way forward marker

**Location:** `internal/websterengine/beginbatch.go:250`, `internal/websterengine/runlevel.go:466`
**Issue:** Both messages name rebaseline in a plain clause ("if the edit keeps every begun batch's cards, run …").
Card 5 is under the way-forward-phrasing decision, whose one form is a trailing `way forward:` clause.
**Fix:** Introduce the existing text with `way forward:`.

### [LOW] Inaccurate new websterengine package documentation

**Location:** `internal/websterengine/doc.go:142–150`
**Issue:** "(RerunCardVerifies)" names a function that does not exist; the helper is the unexported `rerunCardVerifies`.
"a fork write outside the worktree's own cards" is given as a correctness example, but `ClassifyViolation` classes only `fork-contract-write` as a fork correctness finding.
"The fingerprint refusals in begin-batch and record-batch name it" should read begin-batch and run.
**Fix:** Correct the three statements.

### [LOW] Docs and comments the goto change falsified

**Location:** `internal/shedengine/doc.go:29`, `contracts/specs/loom-status-spec.md:63`, `internal/loomcli/cli.go:382`, `internal/shedcli/table.go:19`, `:29`, `internal/shedcli/doc.go:29`
**Issue:** shedengine's doc still says budget exhaustion "persists ReasonBounceBudgetExhausted".
loom-status-spec still derives the budget from stuck entries "since its own most recent done entry", with no mention of the goto boundary.
Four comments still say "the four generic verbs".
**Fix:** Update each to the goto-era behavior.

### [LOW] Rebaseline refusal with no recorded start commit

**Location:** `internal/websterengine/rebaseline.go:87`
**Issue:** When no batch record carries a StartSHA, the message reads "reset the branch to the run's start commit  with git", with a double space and no commit.
**Fix:** Name the SHA only when one is known.

### [LOW] Mid-sentence hard-wraps in new and changed Go comments

**Location:** for example `internal/shedengine/goto.go:2`, `internal/shedverbs/doc.go:1–4`, `internal/websterengine/recordbatch.go:1–8`, `:137–143`, `:317–319`, `internal/websterengine/audit.go:362–372`, `internal/websterengine/auditledger.go:1–6`, `internal/loomengine/anomaly.go:158–163`, `internal/webstercli/recordbatch.go:9–11`, and many test doc comments
**Issue:** CONSTRAINTS.md's Comment Line-Break Convention asks for one sentence per line, broken further only at independent-clause boundaries.
These comments, added or changed in this diff, break mid-phrase at a fixed column ("the entry / observation", "It owns the five verb / bodies").
The convention covers only new and changed comments, and these are.
**Fix:** Rewrap each added or changed comment block to semantic line breaks.

### [LOW] Fabric-sync way-forward tests do not take the way forward

**Location:** `internal/webstercli/verbs_test.go:1184` and the sibling record-batch and recover-batch tests
**Issue:** Card 24 asks the test to take the way forward so the saved state is committed.
The tests take it by setting `fx.CLI.openFabric = nil`, which makes `fabricSync` return before it commits anything.
They show only that the next verb proceeds, not that the next verb's sync (or `lyx fabric commit`) commits the saved state, which the comments claim.
**Fix:** Take the way forward with a working opener and assert the commit happened, or reword the test comments to claim only what the tests show.

### [NIT] batten goto example names no row

**Location:** `internal/battencli/cli.go:187`
**Issue:** `lyx batten goto some-slug --to Teardown` names no batten row (the row is `Worktree-Teardown`), and the verb's own Long uses the right name.
**Fix:** Use `Worktree-Teardown`.

### [NIT] Duplicate helper and missing file header

**Location:** `internal/websterengine/batchfail.go:103`, `internal/websterengine/rebaseline.go:1`
**Issue:** `containsString` reimplements `slices.Contains`, which rebaseline.go in the same package already imports `slices` for.
rebaseline.go opens straight on `package`, while every sibling file carries a header comment.
**Fix:** Use `slices.Contains`, and add a one-line header.
