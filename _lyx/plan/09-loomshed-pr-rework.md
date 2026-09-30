# Card 9 — loomshed-pr-rework

**Create:**
- `internal/loomshed/prrework.go`
- `internal/loomshed/prrework_test.go`

**Uses:**
- `internal/planparser/appendonly.go`
- `internal/planparser/parse.go`
- `internal/loomshed/planwrite.go`

**Intent:**
Add the rework row's producer (D5): it wraps the gated rework session and does everything Go owns around it.
`PRReworkDeps` carries told values and seams only:
`PlanDir`, `ReworkDir` (absolute), `ReworkDirRel` (anchor-relative), `ReadCommitted func(anchorRel string) ([]byte, bool, error)`, `ReadRejection func() (PendingRejection, bool, error)`, `ClearRejection func() error`, `Commit func() error` and `Rebaseline func() error`.
`PendingRejection{PRNumber, HeadSHA, RejectedAt, Findings}` is this package's own view of the record, filled by the caller from `landingshed.ReadRejection`, the way `battenshed.ChildApproval` is filled;
reading through seams keeps `internal/landingshed` off `loomshed`'s import allowlist (`seam_enforcement_test.go`).
`NewPRRework(name string, inner shedengine.ShedProducer, deps PRReworkDeps) shedengine.ShedProducer` returns an unexported `prRework`, a distinct type so the recipe shape test can tell the row apart.
`Call`, in order:
1. `entryErr`; read the pending record through `ReadRejection`; absent is `Stuck` telling the operator to run `lyx loom reject <review-file>`, and a read error is `Stuck` carrying it, so a malformed record halts for a human who re-runs `reject`.
2. Re-entry: when a `round-<N>` directory under `ReworkDir` has a `record.json` committed at HEAD (read through `ReadCommitted`) whose `head_sha` equals the pending record's, skip to step 6.
3. Call `inner`; any error or non-`Done` verdict is returned unchanged, with nothing committed, so a failed attempt's edits stay for the next attempt's check.
4. Append-only check: parse the committed plan with `planparser.ParsePlanFrom` over `ReadCommitted(planparser.PlanDirRel()/<name>)`, parse the working tree with `planparser.ParsePlan`, and return `Stuck` with the joined `planparser.CheckAppendOnly` violations when any exist.
   Baselining on HEAD, not a pre-session snapshot, keeps an edit left by an earlier `Stuck` attempt caught on the retry.
5. Write round N: N is the working-tree round directory whose `record.json` already carries this `head_sha` (a crash before the commit), else the highest round plus one;
   write `findings.md` (the findings), `record.json` (`pr_number`, `head_sha`, `rejected_at`) and `coverage.md` (copied from the output pointer's `Path`), then call `Commit`.
6. Call `Rebaseline`, then `ClearRejection`, then return `Done`.
A `Commit`, `Rebaseline` or `ClearRejection` failure is a returned error, never `Stuck`, for the reason `planWrite.Call` gives: re-running the session cannot fix a git or filesystem fault.
A nil seam is a named error, as `planWrite` guards its commit seam.
`prrework_test.go` (Tier 1, fake inner producer, fake record seams, and in-memory `ReadCommitted` over a map) covers:
an edited existing card, a flipped `approved:`, an edited plan-level section and zero new cards each `Stuck` with a naming reason and no `Commit`;
one appended card `Done` with the round directory's three files, one `Commit`, one `Rebaseline` and the record removed;
re-entry after the commit but before removal completes without calling `inner` and without a second round directory;
a retry after a `Stuck` attempt that edited an existing card still returns `Stuck` while the edit remains;
an absent record `Stuck` naming `lyx loom reject`.
