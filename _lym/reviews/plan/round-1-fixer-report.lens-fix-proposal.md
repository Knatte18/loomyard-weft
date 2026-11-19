```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged the reviewer's proposals that the fixer carried out as written (F1–F3, F6–F15). Two findings.

**MEDIUM — F8, the `TestMain` texts in card 16 (and possibly card 3)**
- **Changed text:** card 16 now says "`TestMain`, in the package's `testmain_test.go`, calls `tmuxkit.Main` as every test package's does". Card 3 has the same sentence for `hardenshed`.
- **Summary:** following the proposal as written breaks the hermetic-git guard for `internal/hardencli`.
- **Evidence:**
  - Card 16's `TestSandboxVerbs` is tier1 "over a hub fixture" and builds wired hubs through `fabriccli.CloneAndWire`. It spawns git, and a hub-fixture token marks a package as git-spawning (`cmd/lym/hermeticenv_test.go:42`, "the hubforge.NewHub token already covers every package that can reach them").
  - `pattern/PATTERN-test-isolation.md:7` says "`TestMain` calls `gitkit.HermeticGitEnv()` before `m.Run()`, or the package is allowlisted".
  - `internal/burlercli/testmain_test.go` is the model to copy: it calls both `gitkit.HermeticGitEnv()` and `tmuxkit.Main`.
  - Card 3's `hardenshed` tests may hit the same guard if `TestReap`'s "unlisted container holding a bare repository" fixture is built with a git spawn.
  - The proposal's "as every other package's does" points at a `tmuxkit`-only shape, like `battenshed`'s, which does not fit a git-spawning package.

**LOW — F3, the deviation bullet in card 16**
- **Changed text:** "`hardencli` uses `internal/hardenshed` and `internal/loomcli`, with no engine package of its own."
- **Summary:** the bullet leaves out packages that card 16's own Intent says `hardencli` calls.
- **Evidence:**
  - Card 16 wires `fabriccli.CloneAndWire`, `fabricengine.CommitAnchoredPaths`, the `configengine` layer write, `shedrun.ReadSeed`, `buildvcs.OfFile` and `selfexec`.
  - Sibling bullets in `pattern/PATTERN-cli-cobra.md` list every package a CLI module calls, for example `adoptcli` (line 62) and `beamcli` (line 63). Against them this bullet is incomplete.

I made 3 tool calls; none edited anything:
- Read `PATTERN.md`'s size, to check F1's 6-byte growth against the overview headroom. It is 9820 bytes, well under the limit, so no finding.
- Grepped `PATTERN-test-isolation.md` for "hermetic", to learn what a test package's `TestMain` must call. The cards and review I had in context did not say.
- Read `PATTERN-test-isolation.md` lines 3–12 and the token lines of `cmd/lym/hermeticenv_test.go`, to confirm how a package counts as git-spawning.
