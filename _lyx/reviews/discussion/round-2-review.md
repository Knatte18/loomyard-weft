---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:98
    summary: D5's Plan severity rule is not a noun phrase, so substituted into D4's mid-sentence slot it renders a malformed boilerplate line, and the acceptance criterion requiring one well-formed sentence cannot be met while D5 is rendered as written
    fix: In D5's Severity rule bullet, end the value at the closing parenthesis, so it reads "a severity (`BLOCKING` when the plan as written fails its verify or ships a wrong behavior, `MEDIUM` when a decision, criterion or branch goes unreached or untested, `LOW` otherwise)", and move the evidence definition into the line D3 adds to every fork's prompt, so D3's fourth bullet reads "Report defects only, never a passed check. Give each finding's evidence as the `path:line` or glyph read." In the acceptance criterion at line 211, replace "as one well-formed sentence" with the exact rendered line - "Return your findings only as your final message, each with a severity (`BLOCKING` when ..., `MEDIUM` when ..., `LOW` otherwise), the section or decision it concerns, a one-line summary and the evidence." - so a test can pin it byte for byte.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:217
    summary: D9 says the bare-token rule reads the whole Type.Method member, but no refuse row pins it, so a check reading only the last segment for both rules passes every criterion
    fix: In the fabric-vocabulary-name refuses criterion at line 217, add the row "a `Type.Method` member under `internal/` whose type segment alone carries a side token".
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:203
    summary: The Open risks fork give-up entry repeats D2's last bound sentence word for word
    fix: Delete the fork give-up bullet at line 203 from Open risks; D2's sentence at line 62 already states the risk and its bound.
---

### [BLOCKING] Plan severity rule breaks the boilerplate sentence it is substituted into

**Location:** `_lyx/discussion/decision-record.md:98`, with D4 at line 86 and the acceptance criterion at line 211.

**Issue:** The fork boilerplate's last line in `contracts/stencils/twill/twill-directive.md` reads "Return your findings only as your final message, each with a severity (`BLOCKING`, `MEDIUM` or `LOW`), the section or decision it concerns, a one-line summary and the evidence."
D4 puts the marker over the parenthesised severity phrase only and asserts that "every host's value reads as a noun phrase in that slot".
Discussion's value does.
D5's Plan value does not: it ends with a second clause, ", with the evidence as the `path:line` or glyph read".
Substituted, the line becomes "each with a severity (…, `LOW` otherwise), with the evidence as the `path:line` or glyph read, the section or decision it concerns, a one-line summary and the evidence."
That sentence nests a second "with" inside the list, so "the section or decision it concerns" now reads as an apposition to "glyph read", and it names the evidence twice.
D5 line 102 says the severity rule is rendered as written, and line 211 requires the twill Plan prompt's boilerplate line to carry the Plan severity rule as one well-formed sentence.
The plan writer cannot satisfy both: rendering D5 verbatim fails the criterion, and rewording it breaks D5.
D4's noun-phrase premise is false for the one non-Discussion host the record defines.
The criterion also leaves "well-formed" to judgment, so no test pins the rendered text.
Round 1's F1 fix proposed this exact value, trailing clause included, so the fixer applied it as given; the slot defect it targeted survives in that clause.

### [LOW] Bare-token reading of a whole `Type.Method` member has no criterion

**Location:** `_lyx/discussion/decision-record.md:217`, against D9 at line 148.

**Issue:** D9 splits the two rules over a method member: the bare-token rule reads the whole member, and the host-identifier rule reads only its last segment.
The refuse rows pin the last-segment case for the host rule ("including as a method's last segment"), but no row has a side token in the type segment only.
A check that reads only the last segment for both rules passes every listed criterion, while the scan, which inspects every identifier including the type name, would refuse the code.

On the rest of this round's D9 focus: each bare-token branch has a row a plan limited to `internal/` would fail (the `Test…` handle outside `internal/` row, the `cmd/` doc-root row), and so does the host rule (the `cmd/` row).
The `tools/` pass row shows a host-geometry identifier outside the scope `hostScope` gives it in `internal/lyxcwd/enforcement_test.go:638` (`!isTest && (inInternal || first == "cmd")`).
D9's host rule matches that scope, including the lack of an owner-set carve-out for the host rule.

### [LOW] Fork give-up risk stated twice

**Location:** `_lyx/discussion/decision-record.md:203`

**Issue:** The Open risks bullet copies D2's sentence at line 62 verbatim, the bound included ("the Plan-Review segment and the plan gates still run on it").
D2 is where the plan writer reads the twill bounds, so the second copy adds nothing and means two edits whenever either changes.
Round 1's F4 fix asked for the sentence in both places; keeping it in D2 alone satisfies that finding's bound requirement.
Verdict on the focus question: the risk does state the bound the Plan-Review segment and plan gates put on it, and its figures hold: `_lyx/config/loom.yaml` sets `plan_timeout_min: 120`, and the boilerplate gives up after 15 `ready: false` waits at a 300000 ms timeout each.

## Focus verdicts with no finding

- D1's rationale matches the code.
  `internal/configengine/config.go` `load` calls `yamlengine.FillMissing`, which appends a template key a present file lacks and never replaces a present value (`internal/yamlengine/fill.go:17`).
  So an absent `plan_producer` takes the template's `single`, and an explicit empty value reaches `loomengine` as `""`, which `discussion_producer`'s switch at `internal/loomengine/config.go:726` refuses, exactly the contrast D1 states.
- D8's bound matches `burlerengine.ResolveFan` (`internal/burlerengine/config.go:111`): it refuses an unknown fan, an empty fan, a fan past `maxClusterN`, and a fan naming an undefined lens, and resolves cfg first with per-name template fallback, so a worktree redefinition changes only the lenses.
- D5 line 102 and D8 line 137 each tell the plan writer, inside `decision-record.md`, that the rejection rule, severity rule, D3's fork line and the lens texts are rendered as written.
