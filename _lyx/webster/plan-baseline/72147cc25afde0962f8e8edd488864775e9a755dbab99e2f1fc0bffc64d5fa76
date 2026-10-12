---
format: 5
approved: true
language: go
---

# Plan: lyx temp root, and `lyx cleanup`

lyx and its tests stop leaving temp garbage in the system temp directory: a new leaf package, `internal/lyxtemp`, owns a per-user root there with one locked run directory per process and a lock-guarded sweep of dead owners' run directories, every lyx temp creation moves into it, every test package runs through `tmuxkit.Main`, and a scan fails a new bare creation.
Then one operator verb, `lyx cleanup`, runs the routine cleanups of the hub prime's leftover state, a dry run by default: the new `lyx batten cleanup` and `lyx orch cleanup`, `lyx board prune`, its own `.scratch/` segment, logger's retention sweep and the temp root's sweep.
The cards follow the decision record's D1–D11; each moves or adds one surface with its tests and the docs it touches.

## Card Index

1 — lyxtemp-package — add the temp root, locked run directories, the sweep and the process API
2 — tmuxkit-run-directory — run tmuxkit.Main in the process run directory and remove it after every run
3 — gitkit-hermetic-run-directory — write the hermetic gitconfig and repo template into the run directory, with the end-to-end test
4 — test-kit-temp-sites — move hubforge, lyxbin, ide and reed test temp directories into the run directory
5 — gate-prebuild-run-directory — build the gate's prebuilt lyx in the run directory and remove it in lyx's main
6 — testtiming-run-directory — move testtiming's directories and attribute leftovers by an inherited marker
7 — testmain-internal-a-l — give internal test packages a to l a TestMain calling tmuxkit.Main
8 — testmain-internal-m-z — give internal test packages m to z a TestMain calling tmuxkit.Main
9 — testmain-kits-tools-contracts — give kit, tools and contracts test packages a TestMain, and gitoracle the kit call
10 — tmux-isolation-every-package — require tmuxkit.Main in every test package
11 — tmux-isolation-guard-rename — rename the tmux-isolation guard after its wider rule
12 — bare-temp-scan — fail a bare system-temp creation outside lyxtemp, with its PATTERN entry
13 — logger-sweep-dry-run — give logger's sweep a dry run and loggerconfig the exit sweep's bound choice
14 — shedrun-cleanup-paths — add the run-records roots and the run directory's relative path
15 — fsx-newest-modtime — add the newest-modification-time helper both age rules read
16 — batten-cleanup — add lyx batten cleanup over done runs and aged orphan scratch
17 — batten-done-refusal — name lyx batten cleanup --apply in the done-slug refusal
18 — orch-cleanup — add lyx orch cleanup, keeping the handoff LastHandoff names
19 — cleanup-scratch-segment — start the cleanup module with its .scratch segment
20 — cleanup-verb — add lyx cleanup over every module's cleanup, mounted in the root

## Shared Decisions

### Decision: temp-root-names

- **Decision:** The root is `<system temp>/lyx-<uid>` off Windows and `<system temp>\lyx` on Windows; a run directory is `os.MkdirTemp(root, "r")` holding its lock file `run.lock`; the creation grace is 10 minutes.
- **Rationale:** `tmuxkit`'s deepest socket path, `/tmp/lyx-1000/r<10 digits>/tmux-1000/<61-byte key>`, is 97 bytes, under `checkSocketPath`'s 107-byte limit, and Windows' temp directory is already per user.
- **Applies to:** cards 1, 2, 12

### Decision: inherited-run-directory

- **Decision:** A process whose system temp directory is a run directory directly under a lyx root takes that root as its own and creates its run directory beside the parent's; no code hands a child a `TMPDIR` that is not a run directory.
- **Rationale:** a child given any other `TMPDIR` would nest its own root and run directory below it and push a socket path past the limit; this is why card 6 drops testtiming's per-package `TMPDIR` and attributes leftovers by an inherited environment marker instead, the one place the decision record's D5 needed a mechanism of its own.
- **Applies to:** cards 1, 2, 3, 6

### Decision: dry-run-shape

- **Decision:** Every cleanup has `lyx board prune`'s shape: one function takes `apply`, computes its candidates the same way in both modes, and removes only when `apply` is true; a dry run creates no lock file and removes nothing.
  The module function holds every refusal and skip, and its verb and `lyx cleanup` add only flag parsing, the prime check and output.
- **Rationale:** a dry run then lists exactly what `--apply` removes, and `lyx cleanup` can never remove more than a module's own verb would.
- **Applies to:** cards 1, 13, 16, 18, 19, 20

### Decision: test-budget-rows

- **Decision:** Every card adding a top-level test, or a new test package, edits `cmd/lyx/testdata/test-budget.yaml`'s row for that package in the same commit; a card that only adds rows or assertions to existing tests leaves it alone.
- **Rationale:** every touched package's row equals its current count, so any new top-level test fails `TestTestBudget_PackagesStayWithinBudget` without its row.
- **Applies to:** cards 1, 2, 3, 12, 13, 15, 16, 18, 19, 20

## Rename mechanic

1. Run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
