# Card 2 — snapshot-title-form

**Create:**
- `plan:internal/alignment#DecisionNoteTitle` -> `func DecisionNoteTitle(slug string) string`

**Edit:**
- `internal/loomcli#alignmentSnapshot`
- `internal/loomcli#TestParentReviewAligner`
- `internal/loomcli#decisionWayNoRecord`
- `internal/loomcli#TestDecisionVerb_Refusals`
- `internal/landingshed#AlignmentDeps`
- `internal/loomcli/doc.go`
- `internal/alignment/doc.go`

**Intent:** D3: a board note counts toward the decision snapshot only by its title.
`DecisionNoteTitle` in `internal/alignment/records.go` returns the title prefix a decision note for a held task starts with, `"Decision for " + slug + ":"`: the slug followed by a colon, so `Decision for foo:` never matches slug `foo-bar`.
Its doc comment says it is the one declaration of the form, that the snapshot filter and the orch role render both call it, and that the match is case-sensitive.

`alignmentSnapshot` keeps hashing the decision record's bytes, and of the board entries keeps only those whose `Kind` is `boardengine.KindNote` and whose `Title` starts with `alignment.DecisionNoteTitle(slug)` (`strings.HasPrefix`, case-sensitive); a note naming the slug only in its brief, body or elsewhere in its title, and an entry of any other kind, no longer counts.
The hashed text of a counted note stays its title, brief and body joined by newlines, sorted by id, so an edit to a counted note still moves the snapshot.
Its doc comment changes from "every note naming slug" to the title form.

`TestParentReviewAligner`'s verdict table, whose fixture slug is `pair`, changes as follows.
The row "an approve after a note naming the slug" flips under D3: its note keeps a title without the form (`hold pair on x`) and gains a brief and a body naming `pair`, and the row now wants `alignment.VerdictApprove` with reason "in scope".
New rows: a note titled `Decision for pair: hold the landing` added after the spawn wants `alignment.VerdictEscalate` with `alignment.DecisionRecordedReason`;
a note titled `Decision for pair-other: x` wants the approve;
an entry of `boardengine.KindTask` titled `Decision for pair: x` wants the approve;
a note titled `decision for pair: x`, the form in another case, wants the approve, pinning the case-sensitive match.
The existing row "an approve after the decision record changed" keeps pinning that a record change still moves the snapshot.

Docs: both places in `internal/loomcli/doc.go` that say the snapshot covers "every board note naming the task" and "the board notes naming the task" say it covers the decision record and the board notes titled `Decision for <slug>:`.
`AlignmentDeps.Snapshot`'s field comment in `internal/landingshed/pralign.go`, "the decision record and the notes naming the task", says the board notes titled `Decision for <slug>:` instead.
`internal/alignment/doc.go`'s `# Decision snapshot` section gains that `DecisionNoteTitle` declares the decision-note title form the snapshot's caller filters on.

The `lyx loom decision add` refusal for a run with no decision record ends with `decisionWayNoRecord`, whose last clause tells the caller to "record the call as a board note naming the task", a note the snapshot no longer counts.
`decisionWayNoRecord` becomes a package-level `var` built from `alignment.DecisionNoteTitle("<slug>")`, so the form keeps one declaration, and its last clause says to record the call as a board note titled `Decision for <slug>: <the call>`, the slug in place of `<slug>`; its other clauses and its two use sites in `checkDarnRecord` and `decisionVerb` are unchanged.
`TestDecisionVerb_Refusals`'s row "darn record missing" wants "board note titled" and `Decision for <slug>:` in place of "board note naming the task".

Bound, per D3: a free-form decision note, including one already on the board for a held task when this lands and one the bouncer escalation stencil counts as an operator decision by its body, no longer turns an alignment approve into an escalation.
Only a note in the title form, or a change to the decision record, does.

**ImpactSummary:** `alignmentSnapshot` has two callers, `newParentReviewAligner` and `landingAlignmentDeps`, both unchanged in shape; only which board notes the hash covers narrows, pinned by the aligner table's rows; the decision verb's no-record way forward and one field comment follow the new form.

**Commit:** 2: Count only notes titled "Decision for <slug>:" in the decision snapshot
