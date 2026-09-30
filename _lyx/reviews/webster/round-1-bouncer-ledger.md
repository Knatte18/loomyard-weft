---
round: 1
ledger:
  - key: panebin-comments-fixed-column-wrap
    rounds: [1]
    status: resolved
  - key: spawn-launchstrandlocked-comments-wrap-and-dangling-contrast
    rounds: [1]
    status: resolved
  - key: replacestrand-cleanup-comment-misstates-condition
    rounds: [1]
    status: resolved
  - key: removestrandlocked-and-down-comments-mid-clause-breaks
    rounds: [1]
    status: resolved
  - key: shell-package-comments-fixed-column-wrap
    rounds: [1]
    status: resolved
  - key: docgo-pane-binary-paragraph-wrap
    rounds: [1]
    status: resolved
  - key: test-file-comments-fixed-column-wrap
    rounds: [1]
    status: resolved
  - key: sandbox-howto-participle-break
    rounds: [1]
    status: resolved
  - key: readtrimmed-misnamed
    rounds: [1]
    status: resolved
  - key: writelaunchscript-split-temp-cleanup
    rounds: [1]
    status: resolved
---

Round 1 raised ten findings.
Eight were LOW comment and markdown line-break violations against CONSTRAINTS.md's Comment Line-Break Convention and the repo's markdown rule, and one of those eight also corrected a comment that misstated `ReplaceStrand`'s cleanup condition.
Two were NITs: `readTrimmed` did not trim, and `writeLaunchScript` handled temp-file cleanup two different ways.
The fixer landed one commit per finding.
The re-check on HEAD `1b09fdc95` confirmed all ten are resolved, and the build, vet, unit and integration suites pass.

The code logic was already correct in round 1 and is unchanged apart from F9's rename and F10's cleanup refactor.
That logic covers the script lifecycle (stage on every launch, delete on removal, `Down` and unpersisted adds) and `ReplaceStrand`'s persisted-or-not guard.

One observation is outside this gate's subject and is not a ledger entry.
The pwsh `Source` form, as the plan specifies it, reads a script without a BOM through `Get-Content -Raw` with no `-Encoding`.
If the pane shell is Windows PowerShell 5.1, that decodes as ANSI, so a non-ASCII path or command would be garbled.
This belongs to the plan.
