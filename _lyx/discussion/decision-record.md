# Publish review fix-back

## Goal

Give loom a path back from a reviewed pull request into the run.
An operator who rejects the open PR with a review file gets the findings planned as new cards, implemented by Webster through the normal fork/record/audit path, re-verified, re-reviewed, re-described, pushed to the same PR, and awaiting approval again — with no fix made outside loom.

## Scope

In:

- A new `lyx loom reject <review-file>` verb beside `approve`.
- Splitting today's `Publish` row into `Publish` (sync, verify, push, open or refresh the PR) and a new PR review gate row that owns approval, rejection and the wait.
- A new rework row that turns the recorded findings into cards appended to `_lyx/plan/`, then routes to `Webster`.
- A sanctioned Webster plan-fingerprint re-baseline so Webster resumes over the extended plan, running only the new cards' batches, then the integration suite with triage.
- `Publish` updating the open PR's title and body from the re-run `Describe` output.
- `approve` and `reject` each clearing the other's pending record.
- Batten's awaiting-child watch resuming the child on a rejection exactly as it does on an approval.
- Recipe header comment, `docs/overview.md`, the `ly-drive` skill text, and CLI help updated in the same change.

Out:

- Tenter.
  It stays the behavior-based live-substrate reviewer of the Hardener campaign in `manifest/designs/hardener.md`; its docs are not touched.
- Fetching findings from GitHub review comments, or posting findings to the PR.
  The review file is the only input.
- Re-running the `Plan-Review` segment over rework cards.
- Any change to `shedengine`'s outcome set or routing vocabulary.

## Decisions

### D1 — A smaller reroute, not Tenter

Decision: build the fix-back as two new recipe rows inside loom's existing producer graph; Tenter is unrelated and untouched.

Rationale: Tenter drives a live substrate to harden a module post-loom; PR fix-back is a text-review → fix-card → implement loop that loom's own rows already do.

### D2 — Split Publish; the review wait becomes its own gate row

Decision: `Publish` keeps sync, post-merge verify, push, and PR create-or-refresh, and returns `Done` once an open PR exists at the pushed head.
Its other verdicts are unchanged: `Done` with no GitHub call when no PR is required against the parent branch, `Done` on a merged PR, and `Stuck` on a closed unmerged PR, since Publish's `Stuck` halts for a human.
A new gate row (working name `PR-Gate`, registry engine in `internal/landingshed`) follows it and owns every review decision.
The gate evaluates these rows in order and the first match wins:

| # | PR / record state | Gate verdict |
|---|---|---|
| 1 | No PR required against the parent branch (same `require_pr_to_base` check as Publish) | `Done`, no GitHub call |
| 2 | PR merged | `Done` |
| 3 | No PR found, or PR closed unmerged | `Awaiting`, reason says reopen the PR or abandon the task |
| 4 | Either record unreadable or malformed | `Awaiting`, reason says re-run the verb to overwrite it |
| 5 | Approval record matches PR number, PR head, and local task HEAD | `Done` |
| 6 | Rejection record matches PR number, PR head, and local task HEAD | `Stuck` → bounces to the rework row |
| 7 | Approval or rejection record present but stale (head or PR number moved) | `Awaiting`, reason names the mismatch and says to re-run `approve` or `reject` |
| 8 | No record | `Awaiting`, reason names the PR URL and both verbs |
| — | Any non-transient failure (origin URL unusable, GitHub client unavailable, GitHub error) | `Awaiting` with the failure as reason; transient failures stay returned errors per the Transient Stop Invariant |

The gate never returns `Stuck` for anything but row 6, because its `Stuck` always routes to the rework row.
`Publish` loses its approval check entirely; the gate is the only reader of the approval record.
Publish's `Finalize` wiring is replaced: `Publish.on_done` names the gate, and the gate's `on_done` names `Finalize`.

A run halted at the gate re-runs only the gate on `lyx loom start`, never `Publish`, so a fix the operator commits by hand outside loom is no longer pushed by a plain `loom start`.
Such an operator pushes it themselves before `approve` (which refuses on the HEAD mismatch until then), or uses `reject`.
`docs/overview.md` and the `ly-drive` skill state this.

Rationale: the recipe's own rule is that a gate bounces to the producer of what it guards; once a producer (the rework row) exists for the diff, the review wait is a gate.
Splitting keeps Publish's real failures (merge conflict, verify failure, push rejected) escalating blocked, while a rejection bounces — both with the existing `Done`/`Stuck`/`Awaiting` vocabulary and no engine change.
Every non-rejection halt at the gate is a human hand-off at the PR, so `Awaiting` is its honest state, and `Stuck` at the gate means exactly one thing: rejected.

### D3 — Segment and budget

Decision: the gate and the rework row share a new segment (working name `PR-Review`); the gate carries `on_stuck: <rework row>` and `max_bounces: 5`.
The rework row carries no `on_stuck` (a writer row whose failure halts for a human, like `Plan-Write`), and `on_done: Webster`.
The gate is on the main line; the rework row is reached only through the gate's `on_stuck`.
The gate returns `Done` only on approval or merge, so its episode `Stuck` count never resets between rounds: the budget is five rejection rounds per task.
Once five rounds have run, `reject` refuses (D4), and the run stays awaiting at the gate, where `approve` is the only way forward.

Rationale: `shedengine` requires an `OnStuck` target to share the bouncing row's segment; five matches the other review segments.
Because `reject` refuses before a sixth `Stuck` exists, the engine's own budget block at the gate is only a backstop.

### D4 — `lyx loom reject <review-file>`

Decision: a new verb in `internal/loomcli`, shaped like `approve`:

- Exactly one positional argument, the review file path; any markdown.
- Refusal table, in order: no status file; run not `awaiting` or `blocked` at the gate row, and not `blocked` at the rework row; at the gate row, the `PR-Review` segment's bounce budget exhausted, meaning `shedengine.Routing.Bounces` reports count >= budget (message names `approve` as the only way forward); origin URL unusable; no PR or PR not open; local task HEAD differs from the PR head; review file unreadable or empty after trimming whitespace.
- Removes any pending approval record, then writes a rejection record `{pr_number, head_sha, rejected_at, findings}` with the review file's full content inline, atomically, to a new loomengine scratch accessor beside `LoomApprovalPath` (ephemeral, under `.lyx`).
  A crash between the two leaves no record rather than two.
- At a run blocked at the rework row, the new record replaces the pending findings; the budget is not consulted, since the gate authored no new `Stuck`.
- Never resumes the run; the envelope carries `pr_number`, `pr_url`, `head_sha`, and `resume: "lyx loom start"`.

`approve` accepts a run awaiting or blocked at the gate row instead of `Publish`, and its help text names the gate.
It removes any pending rejection record before writing the approval, in the same order as `reject`.
The `Rejection` type, its reader and its writer live in `internal/landingshed` beside `Approval`, since the gate reads it.

Rationale: mirrors `approve`'s proven refusal table and never-resume rule, so a supervisor driving `lyx loom step` is never raced.
The latest operator decision wins, and a stale decision can never be acted on because the gate matches both records against the current heads.

### D5 — The rework row

Decision: a new LLM producer row (working name `PR-Rework`, engine in `internal/loomshed`) run as an interactive single-session producer through the same seam `Plan-Write` uses, with its own stencil `contracts/stencils/loom/loom-template-rework.md`.

- Inputs: the rejection findings, `_lyx/plan/`, `_lyx/discussion/decision-record.md`, and the task diff read with read-only git.
- Output: one or more new card files appended to `_lyx/plan/`, numbered after the highest existing card, plus their Card Index entries.
  Nothing else in the plan may change: no existing card, no frontmatter key (the `approved:` flag included), no plan-level section.
- Every finding must be covered by at least one new card; several findings may share a card.
- The stencil points at `loom-template-plan.md` for the card format and never restates it (Producer Pointer-Rule Invariant).
- Gate: `gate: plan` (the existing `planglyph.ValidateFormat` gate, `gate_attempts: 3`), so Gate Self-Check Parity holds with the existing `validate-plan` verb.
- After the gated session, Go runs an append-only check comparing the plan as committed at HEAD, read with read-only git, against the working-tree plan after the session; any change to a pre-existing card, the frontmatter or a plan-level section, or zero new cards, is `Stuck` with a reason naming the violation.
  Baselining on HEAD rather than a pre-session snapshot keeps a failed attempt's leftover edits caught on the retry.
  The comparison uses `internal/planparser`'s model; any new comparison helper lives in `internal/planparser` or `internal/planglyph`.
- On success, in order: write the findings durably to a per-round directory under loom's durable `_lyx` subtree (round N = 1, 2, …), together with the rejection's `pr_number` and `head_sha`; commit the new cards and the round directory through a weft commit seam; re-baseline Webster's plan fingerprint (D6); delete the pending rejection record; return `Done`.
- Re-entry after a crash: on entry, when a round directory committed at HEAD carries the pending record's `head_sha`, the row skips the session and the commit, re-runs the idempotent re-baseline, deletes the record, and returns `Done`.
- Recovery from a `Stuck` rework row, which blocks the run there: `lyx loom start` re-runs the row with the same findings; `lyx loom reject <review-file>` first replaces the findings (D4).

Rationale: Webster executes cards, not free-form findings, so something must plan them; a dedicated producer reuses Plan-Write's machinery and gate without making Master a planner.
Appending to the one plan keeps one Webster state, one card numbering, and the `N: <name>` commit trail that is the resume mechanism.

### D6 — Webster resumes over the extended plan

Decision: `internal/websterengine` exports a re-baseline function that restamps `state.json`'s plan fingerprint under `AcquireStateMutation`; the rework row calls it as a sanctioned rewrite.
Webster's existing run entry then treats the run as a resume: completed batches are skipped, the new cards' batches run through begin/fork/await/record, the stale outcome, summary and integration report are archived or removed as today, and the integration suite runs with triage.
No Webster run-entry special case is added for rework.

Rationale: the package's own rule is that every sanctioned plan rewrite is re-baselined immediately by the writer; without it Webster refuses with `ErrFingerprintMismatch`.

### D7 — Downstream rows re-run unchanged

Decision: after `Webster`, the main line re-runs as it stands: `Webster-Bouncer`/`Webster-Burler` over the whole task diff, `Describe`, `Publish`, the gate.
`Publish`, finding the open PR, pushes and edits the PR's title and body from the description when either differs, then returns `Done`.

Rationale: the rework diff needs the same review, verify and description as the original; the PR must describe what it now contains.

### D8 — Batten resumes on either operator decision

Decision: batten's awaiting-child handling treats a rejection record as an operator decision exactly like an approval: resume the child once per decision identity, recorded in its one acted-on marker.
The identity is the record kind, its timestamp and its head SHA (for example `approve <approved_at> <head_sha>` and `reject <rejected_at> <head_sha>`), so an approval and a rejection at the same head within the same second never share one.

Rationale: a batten-driven child awaiting at the gate would otherwise sit forever after `reject`.

### D9 — Rework cards skip Plan-Review

Decision: the rework row routes straight to `Webster`, not through the `Plan-Review` segment.

Rationale: the findings are operator-authored and authoritative; the resulting diff still passes `Webster-Review` and the operator's own PR review.

## Constraints

- `shedengine` is unchanged: outcomes stay `Done`/`Stuck`/`Awaiting`, and routing stays per-row `on_done`/`on_stuck` (Shed Producer-Seam Invariant).
- New rows are registered through `internal/shedrecipe`'s registry map and get `internal/loomshed` `Name*` constants pinned by the recipe coverage guard (Shed Recipe Registry Invariant).
- Engines take told paths; the new scratch path is a `loomengine` accessor (Told-Geometry, Durable-vs-Ephemeral State, Cwd Resolution Invariants).
- The rework agent writes `_lyx` files; Go commits them through `fabricengine` via a commit seam (Fabric Git Invariant).
- `internal/planparser` stays the sole parser; Go writes no plan bytes, the agent does (Planparser Sole-Parser Invariant).
- The rework session runs interactive under tmux via shuttle/reed, never `claude -p`.
- `reject` follows the CLI/Cobra Invariant: non-empty `Short`, JSON envelope via `internal/output`, `clihelp.ShouldAbort` first, help-tree tests updated.
- GitHub access goes through `internal/githubclient` (GitHub Auth Invariant).
- Gate verdicts never carry the transient mark (Transient Stop Invariant).
- Docs land in the same commit: recipe header comment, `docs/overview.md` loom/landing/batten paragraphs, `plugins/ly/skills/ly-drive/SKILL.md` wording about resuming after approving, and both places' statement that a hand-made fix is pushed by the operator, not by `loom start` (D2).

## Auto-mode assumptions

- Row and segment names `PR-Gate`, `PR-Rework`, `PR-Review` are working names; the plan writer may pick others that fit the recipe's naming, stated once.
- `max_bounces: 5` for the new segment, matching the other segments.
- The review file is free-form markdown; no schema is imposed on findings.
- A run in flight awaiting at `Publish` when this lands migrates without intervention: `Publish` re-runs, finds the open PR, returns `Done`, and the gate then applies any existing approval.
  An operator who has not yet approved such a run runs `lyx loom start` first, since `approve` now accepts only the gate row.

## Open risks

- A future grouping batchifier could regroup pre-existing cards together with appended ones and renumber batches, invalidating Webster's per-batch state; only the identity batcher exists today.
- `internal/shedcheck`'s validity checks may flag the new off-main-line writer row or the gate's shape; the plan writer confirms the recipe passes it.
- The whole-task `Webster-Review` re-review costs a full review round per rejection.
- The rework agent cannot be mechanically checked for covering every finding, since findings are free-form; coverage rests on the stencil and on `Webster-Review` plus the operator's next PR review.

## Acceptance criteria

- `lyx loom reject <file>` on a run awaiting at the gate with an open PR at the local HEAD writes the rejection record, removes any approval record, and prints the success envelope; it never changes the status file.
- `reject` refuses, with one error envelope each: no status file; wrong row or state; exhausted segment budget (count >= budget); origin URL unusable; no PR; PR not open; HEAD mismatch; unreadable or empty review file.
- `reject` at a run blocked at the rework row replaces the pending rejection record.
- `approve` accepts a run awaiting at the gate row, refuses one at `Publish`, and removes a pending rejection record.
- Gate verdict table (Tier 1, fake GitHub): every row of the D2 table, including the no-PR-required `Done` with no GitHub call, a closed PR outranking a matching approval, approval precedence over a stale rejection, and a merged PR precedence over any record.
- Recipe routing test: `Publish.on_done` → gate; gate `on_done` → `Finalize`, `on_stuck` → rework row; rework `on_done` → `Webster`, no `on_stuck`; both new rows in the same segment; the main line still ends at `Friction-Reflect`.
- Rework append-only check (Tier 1): an edited existing card, a changed `approved:` flag, an edited plan-level section, and zero new cards each produce `Stuck` with a naming reason; one or more well-formed appended cards produce `Done` and the durable findings file.
- After the rework row's re-baseline, a Webster run over the extended plan does not return `ErrFingerprintMismatch`, skips completed batches, and dispatches only the new cards' batches, followed by the integration stage.
- `Publish` with an open PR returns `Done` and edits the PR title and body when the description changed, and leaves them untouched when it did not.
- Batten resumes an awaiting child once for a new rejection record and not again for the same one, and resumes it again for an approval at the same head recorded within the same second.
- A rework row re-entered after its commit but before record deletion completes without appending cards a second time.
- A rework row re-run after a `Stuck` attempt that edited an existing card still returns `Stuck` when the edit remains.
- `go test ./...` passes, including the updated landingshed Publish integration test that previously expected `Awaiting` after PR create.

## Notes for the plan writer

- Existing anchors: `internal/loomcli/approve.go` (verb shape and refusal table), `internal/landingshed/approval.go` and `publish.go` (`checkApproval`, the awaiting paths), `internal/landingshed/doc.go` (package doc to rewrite), `contracts/recipes/loom-recipe.yaml` (header comment enumerates the escalate-to-human rows and must be revised), `internal/loomshed/planwrite.go` and `webster.go`, `internal/websterengine/runlevel.go` (fingerprint handling, `restampAndSaveFingerprint`), `internal/battenshed/innerrun.go` (`callAwaiting`), `internal/loomshed/interruptpolicy.go` (policy per row).
- `docs/overview.md`'s loom and batten paragraphs describe the Publish wait and `approve`; both need the gate and `reject`.
