# Card 14 — loomcli-reject-verb

**Edit:**
- `internal/loomcli/cli.go`
- `internal/loomcli/cli_test.go`

**Create:**
- `internal/loomcli/reject.go`
- `internal/loomcli/reject_test.go`

**Uses:**
- `internal/loomcli/approve.go`
- `internal/landingshed/rejection.go`
- `internal/shedengine/progress.go`

**Intent:**
Add `lyx loom reject <review-file>` beside `approve` (D4), under the CLI/Cobra Invariant.
`reject.go` mirrors `approve.go`'s shape: a thin cobra shell over `rejectVerb(ctx, out, d rejectDeps, reviewFile string) int`, every side effect an injected closure, exactly one envelope emitted.
`rejectDeps` carries `readStatus`, `branches`, `findPR`, `headSHA`, `readReviewFile`, `removeApproval`, `writeRejection`, `routing` (the recipe's `shedengine.Routing`, from `loomrecipe.Routing()`) and `now`.
Refusal table, in order, one error envelope each:
1. no status file;
2. the run is neither awaiting nor blocked at `NamePRGate`, nor blocked at `NamePRRework`;
3. at the gate row only, `routing.Bounces(NamePRGate, st.History)` reports count >= budget, and the message names `lyx loom approve` as the only way forward;
4. origin URL unusable; 5. no pull request; 6. pull request not open; 7. local task HEAD differs from the pull request's head;
8. review file unreadable, or empty after trimming whitespace.
At the rework row the budget is not consulted, because the gate authored no new Stuck.
On success it calls `removeApproval` and then `writeRejection` with `{pr_number, head_sha, rejected_at, findings}` (the file's full content), never touches the status file, never resumes the run, and emits `pr_number`, `pr_url`, `head_sha` and `resume: "lyx loom start"`.
Cobra: `Use: "reject <review-file>"`, `Args: cobra.ExactArgs(1)`, a non-empty `Short`, a `Long` naming the gate and rework rows, the budget refusal and the never-resume rule, `clihelp.ShouldAbort` first in `RunE`;
the fabric, GitHub, `LoomApprovalPath` and `LoomRejectionPath` closures are built as `approveCmd` builds its own.
`cli.go` registers `c.rejectCmd()`, adds `"reject"` to `verbUsesLightweightWiring` for the reason `approve` is there, and names `reject` in the parent `Long` and its examples.
Tests: `reject_test.go` table-drives every refusal (one envelope, no record written), a success at the awaiting gate (approval removed before the rejection is written, envelope keys exact), a success at the blocked rework row replacing the pending record with the budget exhausted, and a budget-exhausted refusal at the gate;
`cli_test.go`'s subcommand list gains `reject`, and its help-tree checks cover the new command's `Short`.

**ImpactSummary:** Adds one subcommand and one lightweight-wiring entry; no existing verb's behaviour changes.
