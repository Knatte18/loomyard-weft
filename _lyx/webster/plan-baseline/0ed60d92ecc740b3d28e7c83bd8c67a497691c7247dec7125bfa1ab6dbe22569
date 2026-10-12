# Card 4 — test-kit-temp-sites

**Edit:**
- `internal/hubforge#buildBareTemplate`
- `internal/hubforge#templateFor`
- `internal/hubforge#TestNewHub_TeardownRemovesJunctionsKeepsTargets`
- `internal/hubforge/doc.go`
- `internal/testkit/lyxbin#buildOnce`
- `internal/idecli#TestMain`
- `internal/ideengine#TestMain`
- `internal/reedengine#TestRemoveSocketFileOnceGone`
- `internal/reedengine#TestRemoveStaleSocket`

**Uses:**
- `internal/lyxtemp#MkdirTemp`

**Intent:**
Move every remaining test-kit and test-file creation in the bare system temp directory into the process run directory, so it goes with the run directory and a killed binary's copy goes to the next start's sweep (D4: once-per-binary fixtures go with it).
Each `os.MkdirTemp("", <pattern>)` below becomes `lyxtemp.MkdirTemp(<pattern>)`, keeping its pattern and its existing removal:

- `hubforge`'s bare template (`buildBareTemplate`) and shape templates (`templateFor`), and the teardown test's container in `TestNewHub_TeardownRemovesJunctionsKeepsTargets`;
- `lyxbin`'s per-ldflags build directory in `buildOnce`, whose doc comment stops naming `os.MkdirTemp("", "lyxbin-")` and names the process run directory;
- the `keybindings-` directory both `idecli`'s and `ideengine`'s `TestMain` create, which they create after `gitkit.HermeticGitEnv()` as today;
- `reedengine`'s short `rg` and `rs` socket directories in `TestRemoveSocketFileOnceGone` and `TestRemoveStaleSocket`, which stay at the depth they had, since under `tmuxkit.Main` the old form already landed in the run directory; their socket key is one byte, far under the limit, so no allowlist entry is needed.

`internal/hubforge/doc.go`'s sentence that the template directory "lives in the binary's temp directory, which tmuxkit.Main removes" names the process run directory instead.
This is a relocation with no new behavior; `hubforge`'s, `lyxbin`'s, `idecli`'s, `ideengine`'s and `reedengine`'s existing tests cover each site, and card 12's scan pins that no bare form returns.

**ImpactSummary:** Five packages' fixture and test temp directories move into the process run directory; paths and removals otherwise unchanged.
