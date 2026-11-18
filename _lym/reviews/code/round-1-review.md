---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: internal/boardengine/intake.go#issueContent
    summary: The board entry's first goal point asks the import to copy the evidence the issue names into the entry or a durable referenced file, and the diff only lists the .lym paths in a note and copies nothing.
    fix: In internal/boardengine/intake.go, or in its caller internal/boardcli/intake.go wherever the worktree root is resolvable, make the import copy each named .lym file it can resolve into the entry body, or into a durable tracked file the entry references. Resolve a path against the worktree of the task slug that the issue's provenance line names. Keep the existing import note for every path that cannot be resolved. Do this for both new notes and folds, and add a row covering it to TestImportIssueNewEntry and TestImportIssueFold. If the run's parent instead accepts flag-only behaviour, leave the code in place and defer this finding to the parent, who records the narrowing with lym loom decision add; the fixer cannot record it.
    origin: lens:code-scope
  - id: F2
    severity: MEDIUM
    class: consistency
    location: contracts/stencils/friction/friction-template-reflection.md:61
    summary: The new advice to attach evidence with --attach contradicts the kept rule that the issue body ends with its provenance line, because --attach appends its sections after the supplied body.
    fix: In contracts/stencils/friction/friction-template-reflection.md, reword the inline-evidence paragraph so the provenance line stays last. One way is to tell the agent to quote the evidence lines in the body before the provenance line. The other is to state that --attach sections follow the provenance line and to relax the ends-with-provenance sentence to allow that. Keep the content-rule sentence about attachments.
    origin: assist:assist
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/boardengine/intake_test.go#TestImportIssueFold
    summary: The fold path's new behaviour has no test, though the summary claims a fold keeps the title. The untested parts are the blank-body note naming the title and the .lym path note inside the folded section, and EmptyBody and LocalPaths on a fold's ImportResult.
    fix: Add a row to TestImportIssueFold that folds an issue with a blank body and a comment naming a .lym path. Assert the full folded body, which holds the empty-body note with the title and the paths note under the From issue heading, and assert res.EmptyBody and res.LocalPaths.
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: design
    location: internal/boardengine/intake.go:154
    summary: The .lym path detector misses real paths and keeps trailing markup. It misses a path right after one of * _ | = or :, such as **.lym/x**, |.lym/x| or key=.lym/x, and any backslash-separated path such as .lym\shuttle\x. It keeps trailing emphasis, giving .lym/x** as the path.
    fix: In internal/boardengine/intake.go, add * _ | = { } to pathStopChars, trim * and _ as well as the sentence punctuation in localPathsNamed, and accept a backslash as a separator around the .lym component in localPathPattern. Do not let foo.lym\ match. Add rows covering an emphasised path and a backslash path to the local-paths row of TestImportIssueNewEntry.
    origin: lens:code-correctness
  - id: F5
    severity: LOW
    class: design
    location: internal/selfreportcli/cli.go:209
    summary: A non-UTF-8 attachment is filed with no error, but the JSON encoding downstream replaces its invalid bytes with U+FFFD, so the filed evidence differs from the file.
    fix: In appendAttachments, refuse an attachment whose content fails utf8.Valid. The error names the file and gives the way forward, attaching a UTF-8 text excerpt. Add a row to TestCommand_RefusedBeforeTransport with an invalid-UTF-8 fixture file.
    origin: assist:assist
  - id: F6
    severity: LOW
    class: consistency
    location: plugins/ly/skills/board/SKILL.md:239
    summary: The skill and the intake help say the named .lym evidence is not on the board, but an issue filed with --attach or quoted lines names the path and carries that evidence inline. The import note itself correctly says only what the issue quotes reached the board.
    fix: In plugins/ly/skills/board/SKILL.md, reword the line to say the named files are not on the machine and must not be searched for, and only what the entry body quotes or attaches of them is available. Make the same change to the matching sentence of the Long help in internal/boardcli/intake.go.
    origin: assist:assist
  - id: F7
    severity: LOW
    class: design
    location: internal/selfreportcli/cli_test.go:337
    summary: The only body-limit refusal uses one file that is over the limit by itself. So nothing pins that the whole rendered body, with the body and headings, is measured in runes, or where the boundary sits.
    fix: In the tables of TestRunCreate_Success and TestCommand_RefusedBeforeTransport, add one row whose body and attachment each fit the limit but whose rendered total is exactly maxIssueBodyRunes, and expect it filed. Add another row whose total is one rune over, and expect it refused. Use multibyte UTF-8 content in one of them so a byte count would fail.
    origin: lens:code-tests
  - id: F8
    severity: LOW
    class: design
    location: internal/boardcli/intake.go:318
    summary: No test pins that the error envelope after a failed comment or close carries empty_body and local_paths, though boardcli/doc.go says it carries the same fields.
    fix: Give the issue in the existing intake import close-failure test a blank body or a comment naming a .lym path, and assert the matching key in the error envelope beside the test's existing assertions.
    origin: assist:assist
  - id: F9
    severity: NIT
    class: design
    location: internal/selfreportcli/cli_test.go#TestRunCreate_Success
    summary: No row covers -b with an empty string together with --attach, so the guard that drops an empty body before the first section is unpinned.
    fix: Add a row to TestRunCreate_Success with -b set to an empty string and one --attach, and expect the body to be the attachment section alone.
    origin: lens:code-tests
  - id: F10
    severity: NIT
    class: design
    location: internal/selfreportcli/cli.go:232
    summary: A base name holding a backtick breaks the single-backtick code span in the Attached heading.
    fix: In attachmentSection, wrap the name in a backtick run one longer than the longest run in the name, padded with spaces when the name starts or ends with a backtick.
    origin: lens:code-correctness
  - id: F11
    severity: NIT
    class: design
    location: internal/boardengine/intake.go:203
    summary: The empty-body note formats the title with %q, so a title holding a quote, a backslash or a non-printable character shows Go escaping in the markdown entry.
    fix: In issueContent, render the title with plain quotes, as in "%s", in place of %q, and keep the blank-body row's expected body in TestImportIssueNewEntry in step.
    origin: lens:code-correctness
---

### [BLOCKING] Import flags named evidence but never copies it

**Location:** internal/boardengine/intake.go#issueContent

**Issue:** The amended board entry's first point reads: "copy the evidence the issue names into the entry, or into a file the entry references that outlives the source worktree".
The diff adds a note listing the `.lym/` paths and a `local_paths` output key, and reads no named file on either the new-note or the fold path.
The summary's "Design decisions" replaces import-time copying with filing-time `--attach`.
Its premise that the worktree is "usually gone" by import time is unmeasured.
The path is not unresolvable either: the issue's provenance line names the task slug, which names its worktree.
The decision record has no `Added after Discussion` entry, and no commit carries a trailer recording the narrowing.
`--attach` helps only issues filed after this change with the flag, not an issue that names a path alone, which is the case the entry describes.

### [MEDIUM] Attaching evidence breaks the provenance-last rule

**Location:** contracts/stencils/friction/friction-template-reflection.md:61

**Issue:** The stencil still says each issue body "ends with a provenance line".
The new paragraph tells the agent to "attach the file with `--attach`", and `appendAttachments` places every section after the supplied body.
A reflection agent that follows both rules files an issue that ends with a code fence, not with the provenance line.

### [MEDIUM] Fold gap handling has no test

**Location:** internal/boardengine/intake_test.go#TestImportIssueFold

**Issue:** `foldIssue` calls `issueContent(issue, "### ")`, and `importIssue` sets `EmptyBody` and `LocalPaths` after either branch, at `intake.go:148-149`.
Every fold row uses a non-blank body without `.lym/` paths, so a change that drops the notes or flags on folds passes all tests.
The summary's claim that a fold keeps the title through the note is untested.

### [LOW] Path detector misses common markdown and backslash forms

**Location:** internal/boardengine/intake.go:154

**Issue:** `pathStopChars` holds only white space, quotes, backticks, parentheses, brackets and angle brackets.
The optional prefix must end in `/`, so `**.lym/x**`, `|.lym/x|`, `key=.lym/x` and `.lym\shuttle\x` produce no match, and neither an import note nor a flag.
`localPathsNamed` trims only `.,;:!?`, so `see .lym/x**` yields `.lym/x**`.

### [LOW] Invalid UTF-8 attachment is altered silently

**Location:** internal/selfreportcli/cli.go:209

**Issue:** `string(content)` keeps invalid bytes, the rune count passes, and JSON encoding replaces them with U+FFFD.
The command reports success for evidence that does not match the file.

### [LOW] Skill and help say inline evidence is absent

**Location:** plugins/ly/skills/board/SKILL.md:239

**Issue:** The skill says the import note "says that evidence is not on the board".
The `intake import` Long help says the same: "since that evidence is not on the board".
The note only says what the issue quotes of them is all that reached the board.
With `--attach`, an issue names the run-directory file and also carries its contents, so the skill's sentence misleads a reader about evidence the entry holds.

### [LOW] Body limit test does not pin the whole-body count

**Location:** internal/selfreportcli/cli_test.go:337

**Issue:** The fixture is `strings.Repeat("x", maxIssueBodyRunes+1)`, over the limit on its own.
A per-file check, a byte count or an off-by-one at the limit would still pass every row.

### [LOW] Close-failure envelope gap keys untested

**Location:** internal/boardcli/intake.go:318

**Issue:** The keys go into `short` before `CommentAndClose`, so the error envelope carries them, as `boardcli/doc.go` says.
No test exercises a gap together with a close failure, so moving the insertions after the call would pass.

### [NIT] Empty body with attachment unpinned

**Location:** internal/selfreportcli/cli_test.go#TestRunCreate_Success

**Issue:** The guard `*body != ""` at `cli.go:201` has no row; dropping it adds two leading newlines before the first heading unnoticed.

### [NIT] Backtick in attachment name breaks heading

**Location:** internal/selfreportcli/cli.go:232

**Issue:** The heading format is ``## Attached `%s` ``, so a name holding a backtick ends the code span early.

### [NIT] Title rendered with Go escaping

**Location:** internal/boardengine/intake.go:203

**Issue:** `%q` turns `Say "hi"` into `"Say \"hi\""` in the entry's markdown note.

## Rejected

- affected-set's LOW on `contracts/orchrole/orch-role.md:347`: the orch's instruction only points at `lym selfreport create`, whose help now carries the hint to carry evidence inline, so that instruction is not left stale.
- assist F2's MEDIUM severity for Windows paths: kept, merged into F4 at LOW, since only backslash forms are missed and forward-slash paths, the form lym prints, all match.
- assist F3's MEDIUM severity for invalid UTF-8: kept as F5 at LOW, since captures and logs are text and the loss is limited to invalid bytes.
