# Card 4 — Run the relevant check in the loom discussion gate and validate-discussion

**Create:**
- `plan:internal/loomshed#ValidateLoomDiscussion` -> `func ValidateLoomDiscussion(decisionRecordPath, supportLogPath, relevantPath string) ([]discussionparser.Finding, error)`

**Edit:**
- `internal/loomshed#NewDiscussionGate` -> `func NewDiscussionGate(decisionRecordPath, supportLogPath, relevantPath string) shuttleengine.Gate`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#resolvePlainGateClosure`
- `internal/shedrecipe#resolveGateSpec`
- `pattern/PATTERN-gate-self-check-parity.md`
- `docs/overview.md`
- `internal/shedrecipe#newTestEnv`
- `internal/testkit/envkit#FullEnv`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.wireLightweight`
- `internal/loomcli#TestWire_DefaultConfig`
- `internal/loomcli#TestWireLightweight_FillsThePathsWithoutLoadingAnyConfig`
- `internal/loomcli#loomCLI.validateDiscussionCmd`
- `internal/loomcli#decisionRecordFixture`
- `internal/loomcli#TestValidateDiscussionCmd`
- `internal/loomcli#TestGateParity_DiscussionGate`
- `internal/loomcli#TestSmokeGate_RepromptsThroughARealPaneAndFixesTheArtifact`
- `internal/loomcli#TestSmokeParentReview_NoticeReachesTheWriterPaneAndApproveLetsTheRunThrough`
- `internal/loomshed#TestNewDiscussionGate`
- `internal/loomshed#TestDiscussionGate_FailureSurfacesItsFindings`

**Uses:**
- `plan:internal/discussionparser#CheckRelevant`
- `internal/discussionparser#Validate`
- `plan:internal/loomengine#DiscussionRelevant`
- `internal/loomshed#writeDiscussionFixture`
- `internal/loomshed#validDecisionRecord`
- `internal/loomcli#darnRecordFixture`
- `internal/loomcli#allDiscussionSectionsContent`

**Intent:**
Wire `CheckRelevant` into the loom discussion gate and the loom branch of `lyx loom validate-discussion` through one package function, per D3 and `PATTERN-gate-self-check-parity`.

`ValidateLoomDiscussion`, beside `ValidateDarnDiscussion` in `internal/loomshed/gates.go`, is the loom run's discussion check: it calls `discussionparser.Validate(decisionRecordPath, supportLogPath)`, returning its error as is, then `discussionparser.CheckRelevant(relevantPath)`, returning its error as is, and returns `Validate`'s findings followed by `CheckRelevant`'s.
`CheckRelevant` runs even when `Validate` returns findings, so one gate re-prompt names every defect.
A missing `relevant.md` adds no finding.

`NewDiscussionGate` takes `relevantPath` and calls `ValidateLoomDiscussion` in place of `Validate`; its outcome mapping, warn line and error handling are unchanged, and its doc comment names the new function.
`gates.go`'s file header pairs `NewDiscussionGate` with `ValidateLoomDiscussion` in its gate-to-verb-function list.
`pattern/PATTERN-gate-self-check-parity.md`'s Discussion-Write and Discussion-Burler-Fix pair names `loomshed.ValidateLoomDiscussion` (`discussionparser.Validate` plus `discussionparser.CheckRelevant`) in place of `discussionparser.Validate`.
`docs/overview.md`'s discussionparser entry adds the relevant list's format to what the package reads and names `ValidateLoomDiscussion` among its consumers, and its `validate-discussion` paragraph says a loom run also checks the relevant list when it exists.
`NewDarnDiscussionGate` and `ValidateDarnDiscussion` are unchanged, so a darn run never reads `relevant.md`.

`shedrecipe.Env` gains a `RelevantPath string` field, documented as the told relevant list path read by the gate resolver's "discussion" gate on a loom run.
`resolvePlainGateClosure`'s "discussion" case, on the loom branch only (after the `DiscussionHistory != nil` darn return), refuses an empty or relative `RelevantPath` with `requireAbsRoot(entry, "RelevantPath", env.RelevantPath)`, a fail-loud wiring guard in `refusal-spec.md`'s sense that no on-disk state reaches, and passes it to `NewDiscussionGate`.
`resolveGateSpec`'s doc comment says "discussion" also requires `env.RelevantPath` on a loom run.
`newTestEnv` and `FullEnv` set `RelevantPath` to `relevant.md` joined onto their temp directory, a file nobody creates, so every existing discussion-gate row still passes.

`loomCLI.wire` and `loomCLI.wireLightweight` set `RelevantPath: loomengine.DiscussionRelevant(location)` beside `SupportLogPath`.
`validateDiscussionCmd`'s non-darn branch calls `loomshed.ValidateLoomDiscussion(c.env.DecisionRecordPath, c.env.SupportLogPath, c.env.RelevantPath)` in place of `discussionparser.Validate`; the darn branch is unchanged.
Its `Long` help says it runs the loom gate's check, the decision record and support log check plus the relevant list check when the list exists, and that a darn run does not read the list.
Its I/O error envelope's prefix becomes `loom: validate discussion: ` in place of naming the decision record path, since the failing read may be the relevant list's; the wrapped error names the path it failed on.
The Ok envelope is unchanged.

`decisionRecordFixture` sets `RelevantPath` to `relevant.md` under `dir` without writing it.

Tests:
- `TestNewDiscussionGate`: every call passes `filepath.Join(dir, "relevant.md")`, which the existing rows never write, so the `Pass` row covers a missing list passing.
  A new subtest writes a record missing `## Goal`, a support log and a `relevant.md` holding one stray paragraph line, and asserts `Passed` is false and `Findings` carries `discussion-section-missing` before the stray-line check name, so `CheckRelevant` runs after `Validate`'s findings and its findings follow them.
  The `AdvisorNotesAreNotRead` comment says the gate reads the decision record, the support log and the relevant list, never an advisor's notes.
- `TestDiscussionGate_FailureSurfacesItsFindings`, `TestSmokeGate_RepromptsThroughARealPaneAndFixesTheArtifact` and `TestSmokeParentReview_NoticeReachesTheWriterPaneAndApproveLetsTheRunThrough` pass a `relevant.md` path beside their support log, never written; their assertions are unchanged.
- `TestWire_DefaultConfig` and `TestWireLightweight_FillsThePathsWithoutLoadingAnyConfig` each gain one assertion, beside their existing `SupportLogPath` assertion, that `c.env.RelevantPath` equals `loomengine.DiscussionRelevant(location)` (`loc` in `TestWire_DefaultConfig`).
- `TestValidateDiscussionCmd` gains three rows: `Findings_RelevantMalformed`, a clean record and support log with a `relevant.md` holding one stray paragraph line, exits 1 with findings containing the stray-line check name;
  `Darn_MalformedRelevantIgnored`, `darnRecordFixture` over a clean record with the same malformed `relevant.md` written to the fixture's `RelevantPath`, exits 0, ok, and commits once;
  and `IOFault_RelevantIsDirectory`, a clean record and support log with `relevant.md` created as a directory, exits 1 through the error envelope, with no findings key, and its message contains `loom: validate discussion: `, the new prefix, which the old `loom: validate discussion record <path>: ` prefix does not contain; the relevant path itself comes from the wrapped read error.
  Its `//testtiming:keep` reason adds the malformed relevant list, its darn disposition and the unreadable relevant list.
- `TestGateParity_DiscussionGate` passes `c.env.RelevantPath` to `NewDiscussionGate` and gains two cases: `Stuck_RelevantMalformed`, the stray-line `relevant.md` (verdict stuck), and `Error_RelevantIsDirectory`, a clean record and support log with `relevant.md` created as a directory (verdict error), so the gate returns `CheckRelevant`'s read error rather than failing as a defect.

**ImpactSummary:** re-signs `NewDiscussionGate`, whose one production caller `resolvePlainGateClosure` and five test callers are on this card; adds an `Env` field filled by both loomcli wire paths and both test env builders; darn gates, `Validate` and `decision add` are untouched.

**Commit:** 4: loom discussion gate and validate-discussion check the relevant list
