# Decision record — deploy-prod-cleanup

## Goal

`update-plugins.sh` (`tools/deploy`) stops pushing the shared `prod` pointer, and its other-lyx-on-PATH warning names only copies of lyx itself, each file once.

## Scope

- `tools/deploy/main.go`: drop the `prod` push and the `prod` ancestry check; keep the clean-tree and pushed-to-`origin/main` refusals.
- `tools/deploy/main.go`: the stray-binary scan skips a binary named `lyx` that is not built from lyx's main package, and reports a file reached through several PATH directories once.
- `tools/deploy/main_test.go`: tests follow the changed behavior.
- `CLAUDE.md` Production section and `README.md`'s production paragraph stop naming `prod`.
- Out of scope: deleting the existing `origin/prod` branch, and the hub-wide orch rules on Bolt.

## Decisions

### Keep the pushed-to-main refusal

Decision: production still refuses a HEAD that is not on `origin/main` after a fetch; only the `prod` ancestry half of the gate goes.
Rationale: `CLAUDE.md` requires a deploy from a clean, pushed `main`, and that guard does not depend on the pointer.

### Recognise lyx by its Go build info

Decision: a PATH binary counts as lyx when `debug/buildinfo` reads it and its main package path is `github.com/Knatte18/loomyard/cmd/lyx`; anything else, a non-Go binary included, is foreign and not reported.
Rationale: build info is embedded in every Go binary and reading it executes nothing, unlike a version probe that would run an unknown program.
Bound: a lyx built from a fork with a different module path is no longer reported.

### Deduplicate by resolved file

Decision: each candidate resolves through `filepath.EvalSymlinks`, and a file whose resolved path was already seen, the production binary's included, is skipped.
Rationale: `/bin` linking to `/usr/bin` names one file twice, and a PATH alias of the Go bin dir names the production binary itself.

## Constraints

- `PATTERN-test-speed` and `PATTERN-test-isolation`: tests stay parallel and keep their temp files under the package's `tmuxkit.Main` run dir via `t.TempDir`.
- `CLAUDE.md` Filesystem links: a test's directory link goes through `internal/fslink`.

## Auto-mode assumptions

- The existing `origin/prod` branch is left for the operator to delete, since deleting a remote branch is outward-facing.
- A foreign binary that shadows the production lyx on PATH gets no new warning; the board entry asks only that foreign binaries not be flagged for removal.

## Open risks

- A Windows PATH alias through a junction resolves only as far as `filepath.EvalSymlinks` follows junctions.

## Acceptance criteria

- A production deploy pushes nothing to origin and prints no `Moved prod` line.
- A HEAD not on `origin/main` is still refused.
- A lyx copy outside the Go bin dir is reported, before or after it on PATH.
- A non-Go file and a Go binary of another main package named `lyx` are not reported.
- One file reached through two PATH directories is reported once, and a PATH alias of the Go bin dir reports nothing.
- `CLAUDE.md` and `README.md` no longer say the script moves `prod`.
