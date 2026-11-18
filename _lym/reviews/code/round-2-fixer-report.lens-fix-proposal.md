```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals for F7 (`doc.go` rewording) and F8 (keep marker), the two findings the fixer fixed as proposed.

**F7 — LOW**
- Changed text: `internal/driftengine/doc.go:276-277`, "A change beside the decision record, such as the plan amendment record or the record's PATTERN changes section, is not exempt and is named as a change."
- The claim holds in code. `internal/driftengine/judgesnapshot.go:338` compares `discussionparser.IdentityText(before)` with `IdentityText(now)`, so a PATTERN edit inside the record is named. The plan amendment record lies outside the loom records dirs, so it is named too.
- The sentence is still loose. "Beside the decision record" does not fit the PATTERN changes section, which lives inside the record.
- It also names "plan amendment record" and "PATTERN changes section", and no other `doc.go` line defines either term. Grep of `doc.go` found `amendment` only in the `scope.json` parent-amendment context. This sentence is the only place the package doc uses these loomcli/loom terms.
- A reader of driftengine alone cannot tell what they are. The wording also repeats a loomcli concept, which is the kind of coupling F7 wanted removed.
- A self-contained form would say the exemption covers only entries and carry-over blocks added to the record, and that any other change to the record or to files outside it is named. That is a wording suggestion, not a defect that ships wrong behaviour.

**F8 — no findings**
- The marker is on the line above `func`, and the `//` separator line matches the sibling convention. The `burlercli/cli_test.go:28-29` and `sheddriver/tacklerprompt_test.go:134-135` lines use the same form.
- The reason text states an assertion the covering test lacks. The review states that `TestDriftDeps_RunWhereTheStepsDirResolves` never asserts `Deps.DecisionRecord`.

Tool calls (1 of 6):
1. `grep` of `doc.go` for `amendment|PATTERN`, plus `IdentityText` in `internal/driftengine/*.go`. My context had only `doc.go` lines 255-299, so I needed to see whether the new sentence's terms are defined elsewhere in the file and where the PATTERN comparison happens.
