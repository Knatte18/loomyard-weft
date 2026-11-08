```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. Below is, per change, the set I found and whether the diff covers it, then the findings.

**Set per change**
- **`selfexec.LymBinary` / `ResolveLymBinary` adopted by pins and spawns.**
  - Covered: `reedengine` `panebin.go`, `lock.go`, `bindings.go`, `spawn.go`, `window.go` and `spawnwatchdog.go`; `loomcli` `cli.go` and `looparm.go`; `orchcli`; `fronttask`; `adoptcli/deps.go`; `battencli` `arm.go`, `cli.go` and `wire.go:1459`.
  - Not covered: the direct `os.Executable` sites in findings 1 and 2.
- **`slackLineExecutable` and `errSlackLineTestBinary` removed.** No reference remains in non-research files.
- **`gitkit.refuseCLIReexec` and `gitkit/reexecguard*` removed.**
  - The only remaining references are in historical `docs/research/**/calls.tsv` files. Covered.
  - The `hermetic.go` doc line was removed, and the `cmd/lym/hermeticenv_test.go` allowlist entry for the old guard test was removed. Covered.
- **`tmuxkit.Main` guard.**
  - Covered: the `tmuxkit` package doc, the `Main` doc and `PATTERN-test-isolation`.
  - Every package calls `Main`, enforced by `cmd/lym/tmuxisolation_test.go` with an empty allowlist.
  - The only packages my scan found without a `tmuxkit.Main` are `tmuxkit` itself and two `planglyph/testdata` fixtures. Covered.
- **`proc.PIDsWithEnv`.** The `proc/doc.go` text covers the Linux/Windows parity. Covered.
- **Process cap.** Covered: the `gatecli/doc.go` "Process cap" section, and `enclosingGateEnv` set in `run.go:151` and read by `runMarker`.
- **PATTERN edits.** `PATTERN.md`, `PATTERN-child-processes`, `PATTERN-leaf-packages` and `PATTERN-test-isolation` were all updated. The prompt shows no declared-changes list, so I raise nothing for that rule. No `Pattern-Departure:` trailer appears in any of the five commit messages.

**Findings**

1. **MEDIUM, design.** The rule and item 2 of the decision record are not met: several lym pins and spawns still call `os.Executable` directly.
   - The rule is `pattern/PATTERN-child-processes.md` (the new bullet, "Only a read of the binary itself ... calls `os.Executable` directly"). Item 2 says "every such pin uses it."
   - Spawns or pins left unconverted:
     - `internal/creelcli/spawn.go:61`: detached `creel daemon`.
     - `internal/gatecli/detachfront.go:158`: detached `gate run-detached`.
     - `internal/reedcli/remove.go:63`: detached `reed remove`.
     - `internal/reedcli/window.go:83`: `lymPath` handed to the terminal opener.
     - `internal/boardengine/spawn.go:32`: `syncExecutable = os.Executable`, which launches `board sync`.
     - `internal/ideengine/spawn.go:225`: stamps the lym path into the `.vscode` config.
   - Most of these have a `testing.Testing()` or suppress guard. Even so, the rule text states no exemption for such a guard.
   - The change description's "Sites left on `os.Executable`" list names none of these six, so they are neither converted nor documented as exempt.
   - Fix: convert them, or amend the rule's bullet to say that a site refusing under `go test` itself may keep `os.Executable`, and list them in the change description.

2. **MEDIUM, design.** The `internal/reedengine/doc.go` addition (around line 399–402) says every pin goes through `ResolveLymBinary`, "even when a test hands the engine `os.Executable`". That is false for two paths:
   - The button prompt re-entry calls `deps.executablePath()` raw at `internal/reedengine/buttonclick.go:156`, then builds the command from `exe`. A test-supplied `os.Executable` would pass through unrefused there.
   - `SpawnWatchdog` takes no injectable lookup. It calls `selfexec.LymBinary()` directly (`spawnwatchdog.go`), so the claim does not apply to it.
   - Fix: wrap line 156 in `selfexec.ResolveLymBinary(deps.executablePath)`, or narrow the doc sentence. The button prompt is a command tmux runs later, so item 2 asks for the wrap.

3. **LOW, consistency.** Four `os.Executable` call sites keep the old reasoning in their comments and have no recorded exemption: `internal/loomcli/drivewiring.go:171`, `internal/battencli/wire.go:1039` and the change description's resolve-once exemption for them. Nothing in the rule or the tests records that this exemption exists, so a later reader sees an unexplained exception to "only a read of the binary itself". The reason belongs in the pattern background, or those two sites should use a resolve-once wrapper over `ResolveLymBinary`.

Tool calls (3 of 6):
1. Grep of stale references to the removed symbols, plus `battencli/wire.go` lines 1030–1060 and its diff: my context lacked how `wire.go` and `cli.go` changed.
2. Diff of `pattern/`, `docs/` and the touched `doc.go` files, plus `reedcli/spawnwatchdog.go`: my context held only the stat for these.
3. `buttonclick.go` lines 145–175, `spawnwatchdog.go` and a grep of `executablePath` uses in `reedengine`: to check whether the "every pin" claim holds on the button and watchdog paths.
