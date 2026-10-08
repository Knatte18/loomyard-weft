---
format: 5
approved: false
language: go
---

# Plan: Webster record-batch and rebaseline: no needless re-drive, a calibrated Merriam base

Webster's record-batch, recover-batch, reset and rebaseline stop forcing a needless re-drive or a dead end on four observed paths.
An abbreviated `head_sha` that names HEAD is resolved to the full SHA (#456, D1).
A stopped-then-resumed fork's report is attributed to its batch (#457, D2).
A Card-Index-only overview change is accepted on a run record without an overview frame hash (#463 rebaseline half, D3).
`tools/tokencount` re-measures Merriam's fixed context from real runs (D4).
Refusal rows and module docs land with the code of each card (D5).

## Card Index

1 — pattern-fill-directive — export the directive fill so a caller with stencil and PATTERN bytes renders it as Directive does
2 — record-batch-resolves-head-sha — record-batch resolves an abbreviated head_sha to the one commit it names, refusing an unresolvable one
3 — recover-and-reset-resolve-head-sha — recover-batch and reset --to report-head resolve head_sha the same way
4 — bracket-transcripts-count — a batch's own bracket transcripts that wrote the report count toward attribution and are re-audited
5 — derived-overview-frame — a state without an overview frame hash derives it from the stored plan baseline copy
6 — tokencount-merriam-start — tokencount measures each Merriam session's start at its prompt Read
7 — tokencount-merriam-fixed-context — tokencount reports a measured Merriam fixed context over the texts MerriamBase counts

## Shared Decisions

- `head_sha` resolution runs through one resolver, `resolveReportHead` in `gitwrap.go`, the one webster file that runs git; every consume site calls it before the value's first use, so the run record never holds an abbreviation.
- The bracket's own transcripts live in their own `BatchState` field, never inferred from `ForkTranscripts`, which begin-batch and recover-batch carry across brackets.
- Fork finding identity is stable across re-reads of a grown transcript: `forkKey` orders by transcript, class and per-class ordinal in transcript order, so the open risk the decision record names is pinned by card 4's tests rather than a code change.
- The recorded overview frame is looked up through `recordedOverviewFrame` with the told `WebsterDir`; no helper derives a path.
- tokencount's two calibration sections reconstruct one set of texts, `merriamTexts`, the texts `MerriamBase` counts but the overview.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
