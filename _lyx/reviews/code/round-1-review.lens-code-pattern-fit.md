```yaml
lens: code-pattern-fit
outcome: reported
```

**Task:** lens code-pattern-fit. I checked the diff against `PATTERN.md`, the `pattern/` background files and `CLAUDE.md`.

**Rules checked**

- **`PATTERN-dobby-starts-runs` (declared add): held.**
  - `PATTERN.md` carries the declared text verbatim.
  - The entry has no background link, and the declaration asks for none.
- **`PATTERN-owner-machine` (declared change): held.**
  - `PATTERN.md` carries the declared text verbatim.
  - `pattern/PATTERN-owner-machine.md` gains a "Passes" section that matches the declaration.
- **`PATTERN-shed-stop-chain` (declared add): held.** The declared text is verbatim in `PATTERN.md`.
- **`PATTERN-cwd-resolution`: held.**
  - `dobbyengine` and `shedarbiter` are added to the bound-packages list.
  - Neither imports `lyxcwd`; `go list` and the grep show no direct import.
  - `shedarbiter/seam_enforcement_test.go` pins an allowlist and denies `lyxcwd` and `*cli` packages.
  - `dobbyengine/seam_enforcement_test.go` exists.
- **`PATTERN-daemon-wakeups`: held.**
  - `pattern/PATTERN-daemon-wakeups.md` lists dobby's loop with the one-tick-a-second floor.
  - `loop.go` enforces `wakeFloor` and doubles up to `idleCeiling`.
  - The batten Run-Shed wait is removed from the list, which fits the removal of `wait()`.
- **`PATTERN-shed-driver`: held.**
  - `pattern/PATTERN-shed-driver.md` is updated, with "A yield is logged at Debug".
  - `sheddriver/drive.go` and `battenshed` log the yield with `logger.Debug`.
  - Dobby calls no step body; it spawns `lyx batten run`.
- **`PATTERN-durable-vs-ephemeral-state`: held.**
  - `lyxdirs.BeamDirName` is added.
  - The only `"beam"` literals outside `lyxdirs` are the cobra `Use`, the `beam.yaml` module name and the argv `beam dobby`. None names the directory.
- **`PATTERN-agent-name`: held.**
  - `dobbyProgram = "DOBBY"` is the beam module's constant, in `beamcli/dobbydeps.go:51`.
  - Formation goes through `agentname.FormatScopedProgram`.
- **`PATTERN-spawn-observability`: held.** Dobby's spawn (`dobbyengine/start.go`) and the pass spawn (`beamcli/dobbydeps.go`) both log Info on success and Warn on failure.
- **`PATTERN-run-process-owner`: held.** `beamcli` registers with `runproc.Register` and the new `RoleDobby`.
- **`PATTERN-tolerant-records`: held on the draft key.** `draft_status` is kept through `storeFile` and `Unknown`, and the version is not raised.
- **`PATTERN-config-layers` (D8 copy): held.** The `darn.yaml` copy is a `configsync` key move (`configsync/movekeys.go`, `configreg/keymove.go`).
- **Commit messages and departures: held.** The range has no `Pattern-Departure:` trailer, and I found no departure that needs one.
- **`CLAUDE.md`, docs in the same commit: held.** `doc.go` is present or updated for `dobbyengine`, `shedarbiter`, `beamcli`, `battenshed`, `sheddriver`, `shedengine`, `boardengine` and `adoptcli`. `docs/overview.md` is updated.
- **`CLAUDE.md`, semantic line breaks in `.md` files: not checked.** I made no tool call on this.

**Finding**

**F-A — LOW, `consistency`.**
- **Location:** `internal/shedarbiter/seam.go:6`, `call.go:14`, `prompt.go:10`; `internal/shedarbiter/doc.go`.
- **Summary:** I could not confirm the claim that batten imports `shedarbiter` without linking tree-sitter.
- **Evidence:** `shedarbiter` imports `loomengine`, which imports `planparser`, `pattern`, `lenses`, `discussionparser` and others.
- **What the checks showed:**
  - `go list -deps ./internal/battencli ./internal/battenshed` shows no `sitter` package, so the batten binary does not link it.
  - The tree-sitter modules are all `// indirect` in `go.mod`.
  - The decision record says "batten imports it without linking tree-sitter", and card 25 may have a test that pins this; I did not look.
  - I saw no test pinning the claim in the `shedarbiter` files I read.
- **Fix:** if no test pins the claim, add a deps scan (e.g. in `battenshed`'s seam test) that fails when a tree-sitter package appears in `battenshed`'s or `battencli`'s deps. If one exists, drop this finding.
- **Confidence:** low.

I found no `BLOCKING` or `MEDIUM` pattern-fit findings.

**Tool calls**
1. A grep for `lyxcwd`, `loomengine` and `loomcli` imports, `beam` literals, spawn logging, `DOBBY` and `Pattern-Departure` trailers. My context lacked the import and literal sets for these packages.
2. A grep for tree-sitter references, `loomengine`'s imports, and the `shedarbiter` seam test and `agentname` scoped-program functions. This checked the tree-sitter claim and the `agentname` formation.
3. `go.mod` and `go list -deps` on `battencli` and `battenshed`. This checked the tree-sitter link directly.
