# Card 7 — landingshed-pr-gate

**Edit:**
- `internal/landingshed/deps.go`
- `internal/loomcli/landingdeps.go`

**Create:**
- `internal/landingshed/prgate.go`
- `internal/landingshed/prgate_test.go`

**Uses:**
- `internal/landingshed/rejection.go`
- `internal/landingshed/approval.go`
- `internal/landingshed/pullrequest.go`

**Intent:**
Add the PR review gate producer that owns every review decision after Publish (D2).
`prgate.go` declares `PRGate` (a `shedengine.ShedProducer`) and `NewPRGate(deps Deps) (*PRGate, error)`, which rejects an empty `ApprovalPath`, an empty `RejectionPath` and a nil `TaskHead`, each with its own error.
Its producer-name constant is `prGateName = "PR-Gate"`, the row name the recipe card pins.
`Call` evaluates the decision record's D2 table top to bottom, first match wins:
1. parent branch absent from `Config.RequirePRToBase` → `Done`, no GitHub call;
2. merged pull request (`MergedAt` set, as Publish checks it) → `Done`;
3. no pull request, or closed unmerged → `Awaiting`, reason says reopen the pull request or abandon the task;
4. either record unreadable or malformed → `Awaiting`, reason names the file and says to re-run the verb that writes it;
5. approval matching the PR number, the PR head and `TaskHead()` → `Done`;
6. rejection matching all three → `Stuck`, reason names the PR, the head and that the findings go to rework;
7. a record present but stale → `Awaiting`, reason names the mismatch (PR head, local head, or PR number) and says to re-run `lyx loom approve` or `lyx loom reject`;
8. no record → `Awaiting`, reason ends with the PR URL (`withPRURL`) and names both verbs.
An unusable origin URL, an unavailable GitHub client, or a non-transient GitHub error is `Awaiting` with the failure as reason;
a transient query failure is returned through `transientFailure`, never marked on a verdict (Transient Stop Invariant).
A `TaskHead` error is a returned error.
`Stuck` is reachable only through row 6, since the recipe routes this row's `Stuck` to the rework row;
every exit consults `cancelErr` first, as Publish's `stuckOrCancelled`/`awaitingOrCancelled` do.
In `deps.go`, add `RejectionPath string` beside `ApprovalPath` and rewrite that pair's doc comment: the gate, not Publish, reads both records and `TaskHead`.
`landingDeps` fills it with `loomengine.LoomRejectionPath(l)` in the same card, because `landingdeps_test.go`'s reflection walk fails on any zero `Deps` field.
`prgate_test.go` (Tier 1, fake GitHub server as in `publish_test.go`) covers every D2 row, including no GitHub request on row 1, a closed PR outranking a matching approval, a matching approval outranking a stale rejection, a merged PR outranking any record, a transient query failure returned as an error, and each constructor refusal.

**ImpactSummary:** Adds one Deps field, filled by landingDeps; Publish and Finalize are unaffected.
