# Card 4 — darn-discussion-gate

**Create:**
- `plan:internal/loomshed#ValidateDarnDiscussion` -> `func ValidateDarnDiscussion(decisionRecordPath, supportLogPath string, history func() ([][]byte, error)) ([]discussionparser.Finding, error)`
- `plan:internal/loomshed#NewDarnDiscussionGate` -> `func NewDarnDiscussionGate(decisionRecordPath, supportLogPath string, history func() ([][]byte, error), commit func() error) shuttleengine.Gate`

**Edit:**
- `internal/loomshed#TestNewDiscussionGate`

**Uses:**
- `internal/loomshed#NewDiscussionGate`
- `internal/loomshed#formatDiscussionFindings`
- `internal/discussionparser#Validate`
- `plan:internal/discussionparser#CheckAddedDecisionsKept`

**Intent:** Build the darn run's discussion check and gate (D4): the existing section check, then the history check, and a commit of the two files once both pass.

`ValidateDarnDiscussion`, in `internal/loomshed/gates.go` beside `NewDiscussionGate`, is the one package function the gate and `validate-discussion` both call on a darn run (`PATTERN-gate-self-check-parity`).
It runs `discussionparser.Validate` and returns its findings when there are any, since a missing or section-less record has no history worth comparing yet.
Otherwise it reads the record, calls `history` for every committed version of it, and returns `discussionparser.CheckAddedDecisionsKept(record, versions)`.
A read error other than not-exist and a `history` error are returned errors, as `Validate`'s I/O errors are.

`NewDarnDiscussionGate` returns a `shuttleengine.Gate` that calls `ValidateDarnDiscussion`.
Findings fail the gate with `formatDiscussionFindings` and the same warn line `NewDiscussionGate` logs, and commit nothing.
A clean check calls `commit` and passes; a `commit` error is the gate's returned error wrapped with the decision record path, so the handoff halts as a transient fault instead of passing over an uncommitted record.
A repeat pass over an unchanged record commits nothing new, because the commit seam it is told already treats a clean path as a no-op.
`NewDiscussionGate` is unchanged and stays loom's gate.

`TestNewDiscussionGate` gains darn subtests over its existing `validDecisionRecord` fixture: a valid record whose history holds no added entry passes and commits once; a valid record missing an `Added after Discussion` entry a history version holds fails naming that heading and commits nothing, including when the newest history version is that same dropping copy, as a `lyx fabric commit` of the overwrite leaves it, since the check reads the whole history and not the latest commit; a record missing a section never calls `history`, the section finding itself being the existing subtests' behavior; a `history` error and a `commit` error each come back as the gate's error.

**ImpactSummary:** two new loomshed functions beside the loom discussion gate; no existing caller changes.

**Commit:** `4: loomshed darn discussion gate checks the record's history and commits it`
