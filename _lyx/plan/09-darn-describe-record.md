# Card 9 — darn-describe-record

**Edit:**
- `contracts/stencils/landing/landing-template-describe-darn.md`
- `internal/landingshed#DarnDescribeInputs`
- `internal/landingshed#DarnDescribeSpec`
- `internal/landingshed#TestDarnDescribeSpec`
- `internal/landingshed/doc.go`
- `internal/loomcli#loomCLI.wireDarn`
- `contracts/stencils#wordingClaims`

**Uses:**
- `internal/loomengine#DiscussionDecisionRecord`

**Intent:** Have darn's Describe session read the decision record beside the board entry for the scope and decisions it describes (D7).

`DarnDescribeInputs` gains a required `DecisionRecordPath`; `DarnDescribeSpec` refuses it empty by name and fills it as the `decision_record_path` marker.
`wireDarn`'s `DescribeSpec` closure tells it `loomengine.DiscussionDecisionRecord(location)`, and `wireDarn`'s doc and the header comment of its file, `internal/loomcli/darnwiring.go`, stop saying the darn Describe reads no decision record.

`landing-template-describe-darn.md`'s "What to read" gains the decision record `{{.decision_record_path}}`: its `Added after Discussion` entries amend the board entry and win over its body where they differ, and its other sections are the writer's own account of scope and early calls, read as a claim against the diff.
The header comment's marker list names the new marker.

`TestDarnDescribeSpec`'s stencil fixture gains `record={{.decision_record_path}}`, its composed-prompt subtest wants `record=` plus the told path, and its empty-field map gains `DecisionRecordPath`.
A new `wordingClaims` row for `landing-template-describe-darn.md` over `LandingTemplateDescribeDarn` claims `{{.decision_record_path}}` and `Added after Discussion`.
`internal/landingshed/doc.go`'s Describe paragraph says `DarnDescribeSpec` reads the decision record beside the board entry, the diff and the review perch's run directory.

**ImpactSummary:** darn's Describe prompt gains one required marker; `wireDarn` is its only producer of inputs.

**Commit:** `9: darn's Describe reads the decision record`
