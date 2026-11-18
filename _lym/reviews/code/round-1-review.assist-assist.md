---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: internal/boardengine/intake.go:205
    summary: Import warns about named evidence but never preserves its contents.
    fix: Implement evidence preservation in the import flow for both new notes and folds, copying recoverable named evidence into the entry or a durable referenced file through the repository path and durability APIs. Retain an explicit missing-evidence warning when the source cannot be recovered; do not substitute warning-only behavior for copying available evidence.
    origin: lens:code-scope
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/boardengine/intake.go:158
    summary: Native Windows evidence paths receive neither the import note nor the local_paths flag.
    fix: Recognize both slash and backslash separators around the exact .lym component in issue text, independently of the importing operating system. Extend the existing import table with relative and absolute Windows paths and a nonmatching foo.lym component.
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/selfreportcli/cli.go:209
    summary: Non-UTF-8 attachments can be filed successfully with silently replaced evidence bytes.
    fix: Validate attachment bytes as UTF-8 before converting them into the issue body, refusing invalid encoding with the filename and a way to supply a UTF-8 excerpt. Extend the existing refusal table to assert that such an attachment never reaches transport.
    origin: lens:code-correctness
  - id: F4
    severity: MEDIUM
    class: consistency
    location: plugins/ly/skills/board/SKILL.md:239
    summary: The board skill treats a local path mention as proof that evidence is absent.
    fix: Describe local_paths as a warning about ephemeral file references and instruct the reader to inspect imported inline evidence before concluding that evidence is missing. Align the changed intake help sentence with that distinction.
    origin: lens:code-affected-set
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/boardengine/intake.go:138
    summary: Folded imports have no assertion for the new evidence-gap notes and result fields.
    fix: Extend TestImportIssueFold with a blank issue body and a comment naming a local path, asserting the persisted folded body, retained title, EmptyBody and LocalPaths in the existing behavior test.
    origin: lens:code-tests
  - id: F6
    severity: MEDIUM
    class: design
    location: internal/boardcli/intake.go:325
    summary: The post-write failure envelope has no coverage for its new gap keys.
    fix: Extend TestIntakeImport_CloseFailureReportsShortResultAndClosePayload with a blank issue body and a comment naming a local path. Assert empty_body and local_paths alongside its existing board persistence and recovery payload assertions.
    origin: lens:code-tests
  - id: F7
    severity: MEDIUM
    class: design
    location: internal/selfreportcli/cli.go:212
    summary: Attachment-limit tests do not distinguish the complete rendered body limit from an individual file limit.
    fix: Add rows to the existing CLI success and refusal tables for a complete rendered body exactly at the limit and one rune over. Use individually valid body and attachment inputs whose combined contents and Markdown formatting cross the limit, and include multibyte UTF-8 content to pin rune counting.
    origin: lens:code-tests
  - id: F8
    severity: LOW
    class: consistency
    location: contracts/stencils/friction/friction-template-reflection.md:61
    summary: The recommended attachment invocation conflicts with the retained final-provenance requirement.
    fix: In the reflection stencil, direct the agent to assemble inline evidence before the final provenance line in the body supplied through --body-file, rather than recommending --attach after a body already ending in provenance. Preserve the existing public-content restriction.
    origin: lens:code-affected-set
---

### [BLOCKING] Preserve evidence during import

**Location:** internal/boardengine/intake.go:205

**Issue:** The first goal bullet requires copying named evidence into the entry or a durable referenced file.
The import adds path names and warnings, then carries only the existing issue body and comments.
Neither the new-note path nor the fold reads the referenced evidence, even when it remains available.
The landing summary explicitly substitutes filing-time attachments for import-time preservation.
No amendment or departure trailer authorizes that substitution.

### [MEDIUM] Recognize Windows evidence paths

**Location:** internal/boardengine/intake.go:158

**Issue:** The pattern requires forward slashes around the ephemeral directory component.
An issue naming `.lym\shuttle\abc\startup-capture.txt` produces no match.
The absolute form `C:\work\.lym\shuttle\abc\startup-capture.txt` is missed too.
Both forms therefore lose the persisted warning and the result flag, regardless of the importing machine.

### [MEDIUM] Refuse attachments that cannot survive JSON encoding

**Location:** internal/selfreportcli/cli.go:209

**Issue:** A readable attachment containing byte `0xff` passes the conversion and size check.
The body reaches `selfreportengine.CreateIssue`, then go-github's JSON encoder replaces invalid UTF-8 with U+FFFD.
The command can report success while losing the captured bytes it promised to preserve.
The attachment boundary needs an explicit encoding policy before transport.

### [MEDIUM] Distinguish a path reference from missing evidence

**Location:** plugins/ly/skills/board/SKILL.md:239

**Issue:** The skill says the evidence is not on the board and tells the agent not to search for it.
Detection establishes only that the issue text names an ephemeral path.
An issue can name that path and quote its evidence inline; both are carried into the entry.
The new attachment workflow can produce exactly that case.
The changed intake help similarly describes the evidence as absent without inspecting inline content.

### [MEDIUM] Cover gap handling in folds

**Location:** internal/boardengine/intake.go:138

**Issue:** New-note rows assert gap notes and flags, but `TestImportIssueFold` uses ordinary bodies and comments.
No assertion pins the newly claimed fold behavior, including retention of the title when the issue body is blank.
The existing fold test can cover this without a separate helper-level test.

### [MEDIUM] Cover gap keys after a GitHub close failure

**Location:** internal/boardcli/intake.go:325

**Issue:** The success test checks gap fields, while the failure-envelope test uses an issue without either gap.
It therefore cannot detect dropping the new fields from the recovery response after the board write.
The old persistence and recovery assertions must remain when extending that test.

### [MEDIUM] Pin the aggregate attachment limit

**Location:** internal/selfreportcli/cli.go:212

**Issue:** The only limit refusal fixture already exceeds the limit before adding formatting or a body.
It passes even if a future implementation checks files separately, excludes Markdown overhead, or rejects the boundary.
The complete rendered-body contract needs boundary rows through the existing CLI tests.

### [LOW] Keep provenance last when carrying evidence inline

**Location:** contracts/stencils/friction/friction-template-reflection.md:61

**Issue:** The stencil still requires the issue body to end with its provenance line.
Its new `--attach` recommendation appends fenced sections after that supplied body.
Following the recommendation leaves the issue ending with a code fence instead of provenance.
This is a contradiction introduced by the changed guidance, not a request to alter the answer key.

## Other focus conclusions

The import note and result flags derive from the same unchanged issue value.
The board wrapper refreshes the returned entry without dropping the flags.
The already-recorded branch remains a no-op whose CLI response uses only `recorded`.
Forward-slash paths in ordinary fences and Markdown links are recognized;
`foo.lym/` and `_lym/` do not match the exact directory component.
Nil bodies, stdin bodies, empty attachments, attachment read failures and embedded backtick runs follow the intended flow.
The size arithmetic includes the rendered sections; the finding concerns its test coverage.
The changed assertions retain their predecessors, and no new waits or sleeps appear.
The overview, CLI help and package documentation changes accompany the new flag and output fields.
Commit descriptions match the mechanics; the scope substitution is recorded in F1.

## Validation

Reviewed the committed range `9c5bd13101af1686192cd463afa2b7b583b72697..HEAD` through the supplied diff pieces.
The affected-package test attempt used `lym gate test --wait --tier 0` under a one-hour shell timeout.
It could not acquire a gate slot because the sandbox makes the hub's gate-holder directory read-only.
No test result is claimed from that attempt, and this infrastructure limitation is not a code finding.
