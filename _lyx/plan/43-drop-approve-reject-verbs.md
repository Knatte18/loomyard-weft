# Card 43 — drop-approve-reject-verbs

**Delete:**
- `internal/loomcli/approve.go`
- `internal/loomcli/approve_test.go`
- `internal/loomcli/approve_awaiting_test.go`
- `internal/loomcli/reject.go`
- `internal/loomcli/reject_test.go`
- `internal/loomcli#reworkRowFor`
- `internal/landingshed/approval.go`
- `internal/landingshed/approval_test.go`
- `internal/loomengine#LoomApprovalPath`

**Edit:**
- `internal/loomcli#Command`
- `internal/loomcli#verbUsesLightweightWiring`
- `internal/loomcli#loomCLI.resumeCmd`
- `internal/loomcli#loomCLI.awaitingResumability`
- `internal/loomcli#loomCLI.startCmd`
- `internal/loomcli#loomCLI.reviewCmd`
- `internal/loomcli#TestResumeVerb`
- `cmd/lyx#pathRules`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#stepAwaitingApprovalResumeDoneTeardown_ArchivesTheRunRecords`
- `internal/battencli#stepAwaitingRejectionResumesTheChildOnce`
- `internal/loomengine#TestLandingAndApprovalAccessors`
- `internal/loomcli#TestVerbRefusals`
- `internal/battenshed#awaitingHandOff`
- `internal/battenshed#TestInnerRun_AwaitingWithoutApprovalWaitsExempt`
- `internal/battenshed#TestInnerRun_AwaitingRevivesADeadDriverAndKeepsItsHints`
- `internal/battenshed#TestInnerRun_AwaitingResumesOncePerApproval`
- `internal/battenshed#TestInnerRun_AwaitingAgainAfterResumeGetsReapproveHint`
- `internal/battenshed#TestInnerRun_AwaitingOldLayoutMarkerGetsReapproveHint`

**Extends:**
- `internal/loomcli/doc.go`

**Intent:** `lyx loom approve` and `lyx loom reject` go (D13): no PR-Gate awaits a decision, so the verbs are unknown commands and the cobra tree drops them (`PATTERN-cli-cobra`).
`Command` stops adding them and drops them from its help text and examples, and `verbUsesLightweightWiring` drops them from its verb list.
The approval record goes with them: `landingshed`'s approval record and its accessors, and `loomengine`'s approval path accessor.
The rejection record and its readers stay until the PR-Rework card removes PR-Rework, their last reader; no verb writes the record from here on.
Batten's decision read stops reading the two records: battencli's `ReadDecision` seam reads no approval or rejection record and answers no decision, so batten never resumes an awaiting child on an operator decision that no verb can write; its two integration fixtures that seeded those records, and a pr-gate alignment attempt beside them, stop seeding them, and `cmd/lyx`'s path-rule table drops both paths.
`resume`'s refusal at an awaiting row that is no Bouncer row, and `start`'s and `review`'s help text, stop naming the verbs: an awaiting row now answers through the verb its waiting note names, such as `lyx loom review approve`.
`TestResumeVerb` drops its PR-Gate row.
`TestVerbRefusals` gains rows running `lyx loom approve` and `lyx loom reject` through the loom command tree, each refused as an unknown command.
Batten's `awaitingHandOff`, the instruction an awaiting child's notice carries, drops its PR-Gate clause naming the two verbs and keeps the review escalation's `lyx loom circling` way forward; the awaiting tests that asserted `lyx loom approve` in that notice assert the circling way forward instead.
No sandbox scenario exercises the verbs, so the sandbox suite needs no change (`PATTERN-sandbox-coverage`).

`doc.go` drops the verbs.

**ImpactSummary:** Two CLI verbs and the approval record go; the cobra tree, the lightweight-verb list, two help texts and one refusal change, and no other command calls them.
