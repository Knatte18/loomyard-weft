```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set — what the diff forces outside the lines it touches.

**Per change, the set found**

- **`DobbyLandings` signature** (`string` → `[]string`):
  - Callers are `beamcli/dobbydeps.go` and `beamcli/dobbypairanchor_test.go`; both are updated.
  - Docs are covered: `beamcli/doc.go` is updated and `battencli/cli.go:92` names the function without its signature.
  - Covered.
- **`machineLandings` seam** (`Names` plus a list):
  - Its test in `beamcli/dobby_test.go` is updated.
  - Covered.
- **`machinename.Name` semantics** (machine file before hostname, new errors):
  - About 30 production callers read the name per check and need no edit.
  - Message sites updated: `battencli/arm.go`, `battencli/owner.go`, `beamcli/dobbydeps.go`, `boardengine/ownerset.go`. Their tests are updated.
  - The `machinename` doc is updated.
  - `internal/hubgeom/slackgeom.go:36` still reads correctly.
  - No hostname-fix wording remains in any other production file.
  - **Not covered:** `docs/overview.md:362` (F1).
- **`errRelativeHome` text** (`home directory must be absolute` → `HOME must be absolute`):
  - A grep finds no test or other file asserting the old text.
  - Covered.
- **`githubclient.CredentialDir` and `cacheDir` delegating to `standalonestate.UserConfigDir`:**
  - The githubclient leaf allowlist and its comment are updated.
  - `githubclient/doc.go:26` describes the same rule, which is unchanged.
  - `hubgeom/slackgeom.go` still uses `CredentialDir`, whose output is unchanged.
  - Covered.
- **`PATTERN-leaf-packages`, `internal/machinename` entry:**
  - Landed with the declared text.
  - The `standalonestate` entry still reads "Imports only the standard library", which holds.
  - The PATTERN.md overview line matches the declared text, so no overview edit is needed.
  - Covered.
- **`PATTERN-owner-machine` background:**
  - The machine-file clause is landed.
  - The rename-sweep clause is deferred to the follow-up task, as the summary states.
  - The Enforcement bullet still says the sole-source scan covers the hostname read only. The record says `machinename` "becomes the only reader of the machine file", and no scan enforces that. That is not a rule violation.
  - Covered.
- **Commit messages:**
  - There is no `Pattern-Departure:` trailer, and none is needed.
- **Stencils:**
  - The diff touches none.

**Findings**

- **F1 — LOW, consistency**
  - Location: `docs/overview.md:362`, and `:367` for the `standalonestate` entry.
  - Summary: the package tree still describes `internal/machinename` as "its hostname cut at the first dot and lowercased, the sole reader of the hostname, a stdlib-only leaf".
  - Why it is wrong: the package now reads `machine.json` first and imports `internal/standalonestate`, so the "stdlib-only" part is false.
  - Evidence: `docs/overview.md:362`, against `internal/machinename/doc.go` as the diff leaves it.
  - Fix: reword the line to name the machine file ahead of the hostname rule, and a leaf importing only the standard library and `internal/standalonestate`.
  - Optional: `:367` could add that `standalonestate` now also resolves the per-user config directory.

Tool calls:
1. A grep for `errRelativeHome` text, `DobbyLandings` files, hostname mentions in docs, and `pauserecord` in patterns; my context lacked the full-repo reach.
2. A grep for `XDG_CONFIG_HOME`, `LOCALAPPDATA` and machine-file mentions outside the diff, for other readers of the directory rule.
3. A grep of `DobbyLandings` in `battencli/cli.go`, the `standalonestate` and `githubclient` lines in `docs/overview.md`, and the sole-source test, for other docs the diff makes stale.
