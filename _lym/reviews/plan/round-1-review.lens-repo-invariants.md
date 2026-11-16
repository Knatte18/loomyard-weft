```yaml
lens: repo-invariants
outcome: reported
```

Task: the repo-invariants lens review of plan cards 1–4.

**Per-card invariants**

- **Card 1** (`machinename-hostname-name`)
  - Test seams: held. It adds no struct field, and `TestFromHostname` stays in the tier0 file.
  - Doc text: held. `internal/machinename/doc.go` says the package is the only reader of the hostname and of the machine file, and `HostnameName` keeps that true.
  - Sole-source scan and leaf allowlist: held. `os.Hostname` stays inside the package.
  - Card order: held. The card is self-contained.
- **Card 2** (`owner-replace-set`)
  - Test seams: held. No `//lym:seam` marker or reflective test covers `OwnerRequest`; my grep found none.
  - Doc text, `pattern/PATTERN-owner-machine.md:27`: held. Card 2 lists the file and rewrites the bullet.
  - Doc text, `internal/boardengine/doc.go:183`: held. Card 2 lists it.
  - Doc text, `internal/boardengine/doc.go:185` ("Only `lym adopt` passes Takeover"): held, because the replace set leaves `Takeover` unset.
  - Doc text, `internal/boardengine/doc.go:174` (the commit subject `"board: owner <machine> for <slugs>"`): partly broken. The card's planned doc edits do not name the new `replacing <names>` subject (LOW, below).
  - Doc text, `ownerset.go`'s file header (lines 1–4: "SetOwners records a machine as the owner of named beam projects"): broken. It is not a target and becomes incomplete (NIT, below).
  - The `PATTERN.md` fenced line: held. `pattern-line-unprescribed` requires a card that edits `PATTERN.md` to fence the declared line.
  - Card order: held.
- **Card 3** (`machine-name-verb`)
  - New test files' tiers: held. `testmain_test.go` is a support file with no tier, `cli_test.go` is tier0 and spawns nothing, and `name_integration_test.go` is tier1.
  - Hermetic git and `tmuxkit.Main`: held.
  - Registration oracle and root `Long` modules list (`cmd/lym/clitree_test.go:152`): held. The card mounts the module and adds `machine` to the list.
  - Sandbox Suite Coverage (`cmd/lym/sandbox_coverage_test.go`): **broken**. No card lists it (BLOCKING, below).
  - Card order: held. It uses `HostnameName` from card 1.
- **Card 4** (`owner-sweep`)
  - Test seams: held. It adds no shared struct field.
  - Tier: held, assuming `TestNameSweep` goes into card 3's tier1 file. The card names no file for it, but every other test file in the package carries a different tier or none.
  - Doc text: held. It edits the new `doc.go`.
  - Card order: held. It edits symbols card 3 creates.

**Findings**

1. **BLOCKING** · scope · card 3 (`03-machine-name-verb.md`, Edit `cmd/lym#newRoot`)
   - Summary: mounting the new module `machine` breaks `TestSandboxCoverage_AllModulesCoveredOrExcluded`, and no card covers it.
   - Evidence: the test is tier0 (`cmd/lym/sandbox_coverage_test.go:76-77`) and requires every registered root module to have either a `**Covers:**` scenario in a `tools/sandbox/*SUITE.md` file or an `excludedModules` entry with a reason (`cmd/lym/sandbox_coverage_test.go:1-3`, `:25-63`).
   - Impact: no card targets either place, so card 3's card gate, which runs tier0 over `cmd/lym`, fails.
   - Fix: add one of these to card 3. Either a `tools/sandbox/*SUITE.md` scenario tagged `**Covers:** machine`, or an `Edit` of `cmd/lym#excludedModules` with a reason. A reason would be that the verb writes the per-user machine file and sweeps every registered hub's Bolt.
2. **LOW** · consistency · card 2 (`02-owner-replace-set.md`, `internal/boardengine/doc.go`)
   - Summary: `doc.go:174` documents the owner commit as `"board: owner <machine> for <slugs>"`, and card 2 changes the subject to `board: owner <machine> replacing <names>` when the set is non-empty.
   - Gap: the card's `doc.go` edit names only the Owners exception and the selection rule.
   - Evidence: `internal/boardengine/doc.go:174`; `02-owner-replace-set.md:30` and `:42`.
3. **NIT** · consistency · card 2 (`internal/boardengine/ownerset.go` file header)
   - Summary: the header at `ownerset.go:3` says SetOwners records a machine as the owner of *named* beam projects. After card 2 it also records replace-selected projects and the prime, and the card does not list the header.
   - Evidence: `internal/boardengine/ownerset.go:1-4`.

**Tool calls**

1. Grep for reflective or `//lym:seam` tests over `OwnerRequest`, a read of `internal/machinename/doc.go`, and a grep of cmd/lym tests for module-list assertions. The manifest lacked the seam markers and the machinename `doc.go`.
2. Grep for docs claiming only `lym adopt` takes another machine's owner. I needed every doc text card 2's Intent could falsify.
3. Read of `cmd/lym/sandbox_coverage_test.go`'s header and allowlist. The module-level invariant behind it was not in the manifest.
4. Read of the sandbox test's tier and of the other tests that walk `newRoot()`. I needed to confirm the test runs in tier0, inside card 3's gate.
5. Read of `cmd/lym/registration_test.go`'s header. I needed to confirm the oracle needs only the mount.
