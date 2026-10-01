---
format: 5
approved: true
language: go
---

# Plan: refusal audit — every webster, shed and loom refusal leaves a way forward

Every refusal a `lyx webster`, `lyx shed` or `lyx loom` run can reach must name a way forward: a `lyx` verb, a transition the run takes by itself, or a plain `git` command on the warp.
A guard that does not protect correctness warns and records instead of halting.
Webster's fork audit splits into correctness and policy findings, disposes of each finding once per run, and fails a batch on its merits instead of wedging it (cards 1–16).
Two verbs are added: `lyx webster rebaseline` accepts a mid-run plan edit (cards 4–6), and `lyx shed goto` moves a halted run onto a named row with a fresh segment budget (cards 17–22).
Five sweep cards give the remaining refusals their way forward and reaching tests (cards 23–27).
The refusal table, package docs, ly-drive skill and the new CONSTRAINTS.md invariant follow (cards 28–31).

## Card Index

1 — audit-severity — give every audit finding a stable key and a D4 correctness/policy class
2 — audit-ledger — state.json ledger that dispositions each audit finding once per run
3 — card-verify-rerun — re-run a batch's card verify commands in-process for D5
4 — batch-card-record — record each batch's card set at begin-batch and recover-batch
5 — rebaseline-engine — accept an on-disk plan edit without dropping batch records, and name it in fingerprint refusals
6 — rebaseline-verb — expose rebaseline as `lyx webster rebaseline`
7 — batch-fail-primitive — one primitive that takes a batch terminal-failed with an archived report
8 — record-batch-audit-dispositions — record-batch warns on policy findings and fails the batch on correctness findings
9 — record-batch-unattributable — archive an unattributable report and re-drive the batch via begin-batch
10 — post-batch-failures — fail the batch on its own done-check findings, warn on drift about later cards
11 — recover-failed-batch — recover-batch proceeds from a failed batch and hands the strand its failure digest
12 — webstercli-failure-envelopes — persist and commit failed batches and archived reports before the error envelope
13 — master-stencil-ladder — teach Master the batch_failed and report_archived rungs
14 — summary-audit-warnings — append an Audit warnings section to summary.md
15 — run-exit-dispositions — run-exit audit warns, passes dispositioned findings, or demotes done to stuck
16 — webster-incident-regressions — pin the 2026-09-30 wedges with regression tests
17 — shed-goto-outcome — declare the history-only goto outcome and its segment budget boundary
18 — shed-goto-engine — the status-file mutation behind goto
19 — shed-step-way-forward — name goto from the missing-producer and exhausted-budget stops
20 — shedverbs-goto — the generic goto verb body
21 — goto-subtrees — register goto in the shed, loom and batten subtrees
22 — loom-goto-coherence — loom accepts goto history and names it from Loom-Preflight's half-finished stop
23 — websterengine-refusal-sweep — way forward and reaching tests for webster engine refusals
24 — webstercli-refusal-sweep — way forward and reaching tests for webster CLI refusals
25 — shed-refusal-sweep — way forward and reaching tests for shed refusals
26 — loom-cli-refusal-sweep — way forward and reaching tests for loom's approve, start and commit-records refusals
27 — loom-gate-refusal-sweep — way forward and reaching tests for loom's gate-row stops
28 — refusal-spec — the refusal table contract doc, one section per module
29 — package-docs — webster, shed and loom package docs describe the dispositions and link the table
30 — ly-drive-skill — ly-drive looks up ways forward in the table and names goto
31 — constraints-invariant — Refusal Way-Forward Invariant and goto in the Shed Verb-Set list
32 — loom-preflight-goto-reentry — Loom-Preflight passes a deliberate goto re-entry instead of naming a forward goto
33 — goto-admissible-targets — goto refuses a running run and every target past the current row
34 — goto-way-forward-address — goto way-forward texts name the run-id and the recipe's own missing-status verb
35 — batten-goto-teardown — batten goto reaches Worktree-Teardown only once the child run is done
36 — fabric-reference-mutation-class — a fabric reference whose command can change files is correctness
37 — rebaseline-card-hashes — rebaseline compares each begun card's content, not only its id
38 — run-exit-pending-findings — run-exit correctness findings stay pending and block run entry until accepted
39 — accept-audit-verb — `lyx webster accept-audit` accepts the pending run-exit findings
40 — loom-approve-way-forward — approve's refusals name git push/pull and a backward goto to Publish
41 — master-stencil-done-checks — the Master stencil stops listing later-card drift as a batch_failed done-check
42 — fabric-words-outside-fabric — reword the fabric side names this branch added in tests
43 — refusal-spec-rework — the refusal table, overview and ly-drive skill follow cards 32–42
44 — fabric-reference-allowlist — a fabric reference is policy only when every command is a known read-only program
45 — accept-audit-sync-failure — accept-audit's fabric-sync refusal gets its table row and a reaching test
46 — pending-findings-block-run — pending audit findings block the shed run with the accept-audit way forward instead of failing it

## Shared Decisions

### Decision: way-forward-phrasing

- **Decision:** a message's way forward is a trailing `way forward: <verb | transition | git command>` clause.
- **Rationale:** one form lets every test and the refusal table's Way forward cell match the message the same way.
- **Applies to:** cards 5, 7, 9, 10, 15, 18, 19, 22–27

### Decision: finding-identity

- **Decision:** a fork finding's identity is its `Key` (transcript path, class, ordinal); a parent finding's is `<session id>/<Key>` with ordinals indexed into the transcript-ordered audit slices.
  The ledger in state.json (`auditDispositions`, batch and run-level `auditWarnings`) is keyed by identity.
- **Rationale:** a whole-session parent audit repeats every earlier finding on each record-batch, so a stable identity is what lets it be dispositioned once; parent ordinals restart per Master session.
- **Applies to:** cards 1, 2, 8, 15

### Decision: failed-batch-shape

- **Decision:** a batch rejected on its merits is terminal with digest status `failed` and `reasons`, its report archived by `archiveStaleReport`, returned as `*BatchFailedError` (`ErrBatchFailed`) naming `lyx webster recover-batch NN`; the CLI saves the whole state and emits `batch_failed: true`.
  An unattributable report is archived and returned as `*ReportArchivedError` (`ErrReportArchived`) naming `lyx webster begin-batch NN`; the CLI emits `report_archived: true`.
- **Rationale:** both replace a non-terminal batch with an OK report on disk, the state that made begin-batch, record-batch and recover-batch refuse each other.
- **Applies to:** cards 7–13

### Decision: goto-lives-in-the-status-owner

- **Decision:** the status-file mutation is `shedengine.Goto`; the generic verb body in `internal/shedverbs` only reads `Spec` and calls it.
  `goto` writes `paused`, never `running`, and appends a history-only `goto` entry.
- **Rationale:** shedengine owns every status-file write and the bounce-budget count; `paused` reads as an ordinary resume to loom's crash observation, while `running` with the lock free reads as a crashed driver.
- **Applies to:** cards 17–22

## verify:

go build ./...
go test ./...
go test -tags integration ./...
go vet -tags smoke ./...
