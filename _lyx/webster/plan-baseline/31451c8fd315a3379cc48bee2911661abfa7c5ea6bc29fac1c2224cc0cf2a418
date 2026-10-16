# Card 3 — creel-launchers-and-docs

**Create:**
- `sandbox/posix/creel-suite.sh`
- `sandbox/win/creel-suite.cmd`

**Edit:**
- `docs/sandbox-howto.md`

**Uses:**
- `sandbox/posix/shuttle-suite.sh`
- `sandbox/win/shuttle-suite.cmd`

**Intent:** Give the `creel-suite` subcommand its launchers and name the creel suite in the sandbox how-to (D4, acceptance criteria).

`sandbox/posix/creel-suite.sh` is `sandbox/posix/shuttle-suite.sh` with `SANDBOX-CREEL-SUITE` and "creel" for the shuttle names in its header comment and `creel-suite` for `shuttle-suite` on its `go run ./tools/sandbox -parent "$HOME/Code" ... "$@"` line; it is executable (mode 0755) like its siblings.
`sandbox/win/creel-suite.cmd` is `sandbox/win/shuttle-suite.cmd` with the same substitutions on its `REM` lines and `go run ./tools/sandbox -parent C:\Code creel-suite %*`.

`docs/sandbox-howto.md`:

- The launcher list in the opening paragraph gains `sandbox/win/creel-suite.cmd` after `sandbox/win/shuttle-suite.cmd`, and the POSIX-equivalents note gains `creel-suite.sh` after `shuttle-suite.sh`.
- Section 4c becomes `### 4c. Run the shuttle, creel or burler suite (optional, needs live tmux + logged-in claude)`; its code block lists `sandbox/win/creel-suite.cmd` between the shuttle and burler launchers, the prose names `lyx creel`'s scenarios beside `lyx shuttle`'s and `lyx burler`'s with "each" for "both", and the teardown parenthetical reads "(for the reed, shuttle, creel, and burler suites)".
- `## See also` gains `- [tools/sandbox/SANDBOX-CREEL-SUITE.md](../tools/sandbox/SANDBOX-CREEL-SUITE.md) — the embedded creel test scheme `sandbox/win/creel-suite.cmd`/`sandbox/posix/creel-suite.sh` follows.`, a link `internal/lyxcwd`'s docs link test resolves against the file card 1 created.

**ImpactSummary:** two new launcher scripts and one doc file; no Go code, and the doc's only new link targets a file card 1 created.
