---
verdict: APPROVED
rationale: "Round-1 review found only LOW/NIT comment-break and readability issues (F1-F10); all ten are fixed in one commit each, re-read against CONSTRAINTS.md's Comment Line-Break Convention they comply, and go build/vet/test plus the integration suite (TestLaunchScript_SourcedScriptRunsInThePaneShellScope against live tmux) pass on HEAD 1b09fdc95."
---

Range: `git diff $(git merge-base main HEAD)..HEAD`, with `main` taken from `product.parent` in `_lyx/shed/reed-launch-script/status.json`.
The branch carries cards 1–5 one commit each, then ten fixer commits for F1–F10.

The report has no `## Focus departures` section, so there is nothing to ratify or reject.

## The review

The review is sound.
It covered every round-1 focus directive, and each directive it settled without a finding holds on re-check:

- `ReplaceStrand`'s `postAddSaveFailed && applyErr != nil` guard is correct.
  `reconcileApplyPersistLocked` (`spawn.go`) returns early only before its `SaveState` (list panes, reconcile, list panes after reconcile, apply layout), and after that `SaveState` it returns nil.
  So a non-nil `applyErr` means no save carrying the new strand succeeded.
- A removed GUID left in `reed.json` without a script is allowed by the CONSTRAINTS bullet, because every launch path restages the script before it types the source statement.
- `writeLaunchScript`'s cleanup covers every failure path.
  Since F10 it is a single deferred removal that is skipped once the rename succeeds.

Severity is calibrated.
Comment-break violations of a written CONSTRAINTS rule are LOW, and the misleading `readTrimmed` name and the two cleanup styles are NITs.

## The fixes on HEAD

I re-read every new and changed Go comment and markdown line in the diff:

- `panebin.go`, `spawn.go`, `replace.go`, `strand.go`, `lifecycle.go` and `doc.go`.
- `shell.go`, `posix.go` and `pwsh.go`.
- The five changed test files.
- `CONSTRAINTS.md` and `docs/sandbox-howto.md`.

Each is now one sentence per line.
Every further break falls at a `, so`/`, and` or `;` clause boundary.
The untouched legacy lines in `doc.go`, `panebin.go`'s header and `emptycmd_integration_test.go`'s header are left alone, as the convention requires.
F3's rewritten comment now states the guard's real condition.
F2's rewrite ties the "rather than changing how the pane's shell itself starts" contrast back to the script route.
`git diff --check` is clean.

## Per-card mechanical checks

- **Card 1 (Edit on `Shell`):** `go vet ./...` compiles every package's tests, and it passes.
  So no `Shell` implementer outside `internal/shell` was left without `Source`/`ScriptExt`.
- **Cards 2, 3 and 4 (Edit):** the blast radius matches each ImpactSummary.
  `removeLaunchScripts` returns nothing, and every call site sits beside an existing return or error, so no operation's result or error changes.
- **Cards 2, 3 and 5 (Create):** the reviewer confirmed that no equivalent state-dir subpath constant existed.
  It also confirmed that the near-equivalent `fabricengine` atomic-write helper is excluded by card 2's own "private helper in this file" mandate, so this is not a diff defect.

Verification on HEAD `1b09fdc95`:

- `go build ./...`, `go vet ./...` and `go test` over `./internal/shell/...` and `./internal/reedengine/...` pass.
- `go test -count=1 -tags integration` over the same packages passes.
- `TestLaunchScript_SourcedScriptRunsInThePaneShellScope` ran against live tmux and passed.

## A plan-level note, not a finding

The pwsh `Source` form reads the script with `Get-Content -Raw` and no `-Encoding`, as card 1 specifies.
The script is written without a BOM, byte for byte, as card 2 specifies.
If the pane shell is Windows PowerShell 5.1, it decodes a file with no BOM as the ANSI code page, so a non-ASCII executable path or launch command would be garbled.
The existing `ReadFile` prompt-file idiom has the same behavior.
The diff implements the plan exactly, so this is a question for the plan, and this gate does not flag it.
