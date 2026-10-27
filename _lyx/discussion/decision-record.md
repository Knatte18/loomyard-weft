# Decision record — rename-binary-lym

## Goal

Rename LoomYard's binary from `lyx` to `lym` so it no longer collides with the LyX editor's `/usr/bin/lyx`, and carry the same rename through the hub suffix, the environment variables and the state directory names.

## Scope

- Every tracked text file: `lyx` becomes `lym`, `Lyx` becomes `Lym` and `LYX` becomes `LYM`, which covers the binary, `cmd/lyx`, `binaryName()` in `tools/deploy`, the `-LYXHUB` suffix, the `LYX_*` variables, the `_lyx/` and `.lyx/` directory names, the `ly` plugin's stencils and skills, scripts and launchers.
- Every tracked file and directory whose path holds one of those spellings is moved to the renamed path.
- Dobby's `markers` directory goes: `dobby.log` sits beside `state.json` and `dobby.lock` in `.lym/beam/`, and board-tidy reports go to `.lym/beam/tidy/<timestamp>.md`; `MarkerDirName` is removed.
- Out of scope: migration code, the weft side, the worktree's own `_lyx` and `.lyx` links and the records under them, and the board entry itself.

## Decisions

### D1: one case-mapping rename script does the text and path rewrite

Decision: a Go program under `.scratch/`, run once and never committed, rewrites the three spellings `lyx`, `Lyx` and `LYX` in every tracked text file and moves every tracked path holding them with `git mv`; package names such as `lyxcwd` and identifiers such as `LyxDirName` are renamed with the rest.
Rationale: the operator asked for a scripted rename; mapping every identifier keeps code, docs and tests in one vocabulary, and a mixed `lyx`/`lym` codebase would leave the old name to be renamed again.

### D2: the spelling `LyX` and two file sets are left as they are

Decision: the script never touches the spelling `LyX`, `go.sum`, or `docs/research/`.
Rationale: `LyX` names the LaTeX editor in a deploy test that guards against a foreign binary; `go.sum` holds hashes; `docs/research/` holds dated measurement reports that record past runs under the name they ran with.

### D3: dobby's log and tidy reports get named paths on the geometry

Decision: `Geometry.MarkerDir` is replaced by `LogPath`, `.lym/beam/dobby.log`, and `TidyDir`, `.lym/beam/tidy/`, a tidy report being named `<UTC timestamp>.md`; `MarkerDirName` gives way to a `TidyDirName` constant.
Rationale: the directory never held marker files, and the board entry names these locations.

### Added after Discussion (parent, 2026-10-27): Remove the empty cmd/lyx directory left by the rename

Decision: Remove the empty, untracked cmd/lyx directory the git mv left on disk, then run verify again with all its tiers. The running gate builds a prebuilt binary only where cmd/lyx exists as a directory (internal/gatecli/prebuild.go), so without it the prebuild is skipped and the tier tests build the binary from the branch's own cmd/lym. Add no cmd/lyx shim and skip no tier.

Rationale: The failure is 'no Go files in cmd/lyx' from a leftover empty directory, not from the rename itself; the old gate's stat check skips the prebuild once the directory is gone.

### Added after Discussion (parent, 2026-10-27): Keep the comment reflow in the rename

Decision: The reflow of the comment blocks the rename touched, made with tools/godocreflow so that each comment has one sentence per line, stays in this run's diff. It changes line breaks only, never words, and is in scope.

Rationale: Operator, 2026-10-27: keep them; agents keep getting these line breaks wrong, so fixing them now is welcome.

## Constraints

- PATTERN entries that name renamed packages or variables are renamed by the same script, with their background files.
- No migration: the code knows only the new names.
- `lyx gate test` stays the running binary for this run's own tests.

## Auto-mode assumptions

- A darn run holds no interview; every call above is the writer's own pick from the board entry.
- Tests that read the prebuilt-binary variable under its new name find it unset under the old gate and build the binary themselves, which costs time but not correctness.

## Open risks

- A test that pins a literal path of the running worktree, or a gate key naming `cmd/lyx`, may fail under the old gate; such failures get hand fixes.
- A `lyx` substring inside an unrelated word would be renamed; the survey found none outside the `LyX` spelling.

## Acceptance criteria

- No tracked file outside `go.sum` and `docs/research/` holds `lyx`, `Lyx` or `LYX`, and no tracked path does.
- `go build ./cmd/lym` produces a binary named `lym` and `tools/deploy` installs `lym`.
- Dobby writes its log to `.lym/beam/dobby.log` and a tidy report to `.lym/beam/tidy/<timestamp>.md`, with no `markers` directory.
- The scoped gate run is green.
