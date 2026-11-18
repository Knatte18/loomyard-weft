---
dispositions:
  - id: F1
    disposition: deferred
    reason: The fix needs a parent call the fixer cannot record. Copying named files means resolving each path in a sibling worktree through the issue's provenance line, which the reflection stencil leaves free-form, so no slug can be parsed reliably. It would also copy unreviewed local file content into a board entry. The finding itself says to leave the code and defer to the parent if flag-only behaviour is accepted, and the parent records the narrowing with lym loom decision add.
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed-differently
    reason: The proposal put `_` in pathStopChars, which would truncate real paths such as `.lym/shuttle/startup_capture.txt`. A separate pathLeadChars set (stop characters plus `*`, `_`, `:`) now lets those precede a path without ending one, and trailing `*` and `_` are trimmed. `|`, `=`, `{` and `}` went into pathStopChars as proposed, and a backslash is accepted as a separator.
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed-differently
    reason: The proposed wording "not on the machine" is stronger than the import note, which says "deleted with its worktree", and a worktree may still exist at import time. The skill and help now say the named files are deleted with their worktree, so do not search for them, and that only what the entry body quotes or attaches is available. The ImportResult doc comment in intake.go got the same "quotes or attaches" wording.
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
---

## Changes

- F2, `contracts/stencils/friction/friction-template-reflection.md`: the body rule now gives a provenance line that is the last of what the agent writes, and says the sections `--attach` appends follow it.
- F3, `internal/boardengine/intake_test.go`: `TestImportIssueFold` folds an issue with a blank body and a comment naming a `.lym` path. It asserts the full folded body, `EmptyBody` and `LocalPaths`. It also asserts no flags on the plain fold.
- F4, `internal/boardengine/intake.go`: `pathStopChars` gains `|={}`, `pathLeadChars` is new, `localPathPattern` accepts a backslash separator, and `localPathsNamed` trims `*` and `_`. A row in `TestImportIssueNewEntry` covers emphasised, pipe-delimited, key=value and backslash paths, and `foo.lym\e` stays unmatched.
- F5, `internal/selfreportcli/cli.go`: `appendAttachments` refuses a non-UTF-8 file, naming it and the way forward. A `TestCommand_RefusedBeforeTransport` row covers it.
- F6, `plugins/ly/skills/board/SKILL.md`, `internal/boardcli/intake.go`: the wording described under the disposition above.
- F7, `internal/selfreportcli/cli_test.go`: a success row with a multibyte body and attachment totalling exactly `maxIssueBodyRunes` characters, and a refused row one character over.
- F8, `internal/boardcli/intake_test.go`: the close-failure test's issue names a `.lym` path and the test asserts `local_paths` is in the error envelope and `empty_body` is absent.
- F9, `internal/selfreportcli/cli_test.go`: a success row with `-b ""` and one `--attach`.
- F10, `internal/selfreportcli/cli.go`: `attachmentSection` renders the name through the new `codeSpan`, with `longestBacktickRun` shared with the fence. Success rows cover a name with an inner backtick and one with edge backticks.
- F11, `internal/boardengine/intake.go`: the empty-body note renders the title in plain quotes.

Commits are per fix where the files allow. F3, F4 and F11 share one commit, and F5, F7, F9 and F10 share another, because they edit the same files and partial staging is not available.

A gate finding then flagged a wrapped comment line on `limitFillerContent` in `internal/selfreportcli/cli_test.go`.
It is rewritten as two sentences, one per line, with no behaviour change.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW, `ImportResult` doc comment still says "quotes" alone: fixed.
- fix-consistency, LOW, stencil "last of what you write" looser than "ends with": no change, the fork named a doubt and no defect.
- fix-proposal, LOW, F6 wording "not on the machine" too strong while a worktree may exist: fixed differently as described under F6.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

- F1: see the disposition reason above. The code stays as it is, with the import note listing the named paths for every path it cannot resolve. The parent decides between building the copy and accepting flag-only behaviour, and records a narrowing with `lym loom decision add`.

## Rulings

None.
